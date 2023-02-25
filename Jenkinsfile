def buildAndPush() {
    // env.BRANCH_NAME is set to the git branch name by default
    // env.REGISTRY is the repository url for this pipeline
    // env.GIT_SHORT is the git short hash of the currently checked out repo
    // env.LATEST_BRANCH is the branch name that gets tagged latest
    // env.ARCH is the system architecture. some apps can be generic (amd64, arm64),
    // but apps that compile for specific hardware (like web3-proxy) will need more specific tags (amd64_epyc2, arm64_graviton2, intel_xeon3, etc.)

    // TODO: check that this system actually matches the given arch
    sh '''#!/bin/bash
        set -eux -o pipefail

        [ -n "$GIT_SHORT" ]
        [ -n "$GIT_SHORT" ]
        [ -n "$REGISTRY" ]
        [ -n "$ARCH" ]

        # deterministic mtime on .git keeps Dockerfiles that do 'ADD . .' or similar
        # without this, the build process always thinks the directory has changes
        git restore-mtime
        touch -t "$(git show -s --date=format:'%Y%m%d%H%M.%S' --format=%cd HEAD)" .git

        function buildAndPush {
            image=$1
            buildcache=$2

            buildctl build \
                --frontend=dockerfile.v0 \
                --local context=. \
                --local dockerfile=. \
                --output "type=image,name=${image},push=true" \
                --export-cache type=s3,region=us-east-2,bucket=llamarpc-buildctl-cache,name=${buildcache} \
                --import-cache type=s3,region=us-east-2,bucket=llamarpc-buildctl-cache,name=${buildcache} \
            ;
        }

        BUILDCACHE="${REGISTRY}:buildcache_${ARCH}"

        # build and push a docker image tagged with the short git commit
        buildAndPush "${REGISTRY}:git_${GIT_SHORT}_${ARCH}" "${BUILDCACHE}"

        # push an image tagged with the branch
        # since buildAndPush just ran above, this should be very quick
        # TODO: maybe replace slashes in the name with dashes or underscores
        buildAndPush "${REGISTRY}:branch_${BRANCH_NAME}_${ARCH}" "${BUILDCACHE}"

        if [ "${BRANCH_NAME}" = "${LATEST_BRANCH}" ]; then
            buildAndPush "${REGISTRY}:latest_${ARCH}" "${BUILDCACHE}"
        fi
    '''
}

pipeline {
    agent any
    options {
        ansiColor('xterm')
    }
    environment {
        // AWS_ECR_URL needs to be set in jenkin's config.
        // AWS_ECR_URL could really be any docker registry. we just use ECR so that we don't have to manage it
        REGISTRY="${AWS_ECR_URL}/erigon"

        // branch that should get tagged with "latest_$arch" (stable, main, master, etc.)
        LATEST_BRANCH="stable"

        // non-buildkit builds are officially deprecated
        // buildkit is much faster and handles caching much better than the default build process.
        DOCKER_BUILDKIT=1

        GIT_SHORT="${GIT_COMMIT.substring(0,8)}"
    }
    stages {
        stage('build and push') {
            parallel {
                stage('build and push amd64 image') {
                    agent {
                        label 'amd64'
                    }
                    environment {
                        ARCH="amd64"
                    }
                    steps {
                        script {
                            buildAndPush()
                        }
                    }
                }
            }

        }
        stage('create (experimental) manifest') {
            agent any
            steps {
                script {
                    sh '''#!/bin/bash
                        set -eux -o pipefail

                        [ -n "$BRANCH_NAME" ]
                        [ -n "$GIT_SHORT" ]
                        [ -n "$LATEST_BRANCH" ]
                        [ -n "$REGISTRY" ]

                        function manifest {
                            repo=$1

                            docker manifest create "${repo}" --amend "${repo}_amd64" --amend "${repo}_arm64"

                            docker manifest push --purge "${repo}"
                        }

                        manifest "${REGISTRY}:git_${GIT_SHORT}"
                        manifest "${REGISTRY}:branch_${BRANCH_NAME}"

                        if [ "${BRANCH_NAME}" = "${LATEST_BRANCH}" ]; then
                            manifest "${REGISTRY}:latest"
                        fi
                    '''
                }
            }
        }
    }
}
