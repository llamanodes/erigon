@Library('jenkins_lib@main') _

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
        stage('push latest') {
            parallel {
                stage('maybe push latest_amd64 tag') {
                    agent any
                    environment {
                        ARCH="amd64"
                    }
                    steps {
                        script {
                            maybePushLatest()
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
