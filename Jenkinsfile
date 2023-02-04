def amd_image
def arm_image
def intel_image
def restoreMTime() {
    sh '''
        git restore-mtime
        touch -t "$(git show -s --date=format:'%Y%m%d%H%M.%S' --format=%cd HEAD)" .git
    '''
}


pipeline {
    agent any
    environment {
        DOCKER_BUILDKIT=1
        DOCKER_REPO="${AWS_ECR_URL}/erigon"
        DOCKER_REPO_TAG_GIT="${DOCKER_REPO}:${GIT_COMMIT.substring(0,8)}"
        DOCKER_REPO_TAG_LATEST="${DOCKER_REPO}:latest"
        PRODUCTION_BRANCH="stable"
    }
    stages {
        stage('build and push') {
            parallel {
                stage('build and push amd64 image') {
                    agent {
                        label 'amd64'
                    }
                    steps {
                        script {
                            ARCH="amd64"

                            restoreMTime()
                            DOCKER_REPO_TAG_LATEST_ARCH="${DOCKER_REPO_TAG_LATEST}_${ARCH}"
                            DOCKER_REPO_TAG_GIT_ARCH="${DOCKER_REPO_TAG_GIT}_${ARCH}"
                            try {
                                docker.pull("$DOCKER_REPO_TAG_LATEST_ARCH")
                            } catch (e) {
                                echo "fallible ${ARCH} pull failed: ${e}"
                            }
                            try {
                                amd_image = docker.build("${DOCKER_REPO_TAG_GIT_ARCH}", "--cache-from=${DOCKER_REPO_TAG_LATEST_ARCH} .")
                            } catch (e) {
                                def err = "${ARCH} build failed: ${e}"
                                error(err)
                            }
                            amd_image.push()
                            if (env.BRANCH_NAME == "$PRODUCTION_BRANCH") {
                                amd_image.push("${DOCKER_REPO_TAG_LATEST_ARCH}")
                            }
                        }
                    }
                }
                stage('Build and push arm64 image') {
                    agent {
                        label 'arm64'
                    }
                    steps {
                        script {
                            ARCH="arm64"

                            restoreMTime()
                            DOCKER_REPO_TAG_LATEST_ARCH="${DOCKER_REPO_TAG_LATEST}_${ARCH}"
                            DOCKER_REPO_TAG_GIT_ARCH="${DOCKER_REPO_TAG_GIT}_${ARCH}"
                            try {
                                docker.pull("$DOCKER_REPO_TAG_LATEST_ARCH")
                            } catch (e) {
                                echo "fallible ${ARCH} pull failed: ${e}"
                            }
                            try {
                                arm_image = docker.build("${DOCKER_REPO_TAG_GIT_ARCH}", "--cache-from=${DOCKER_REPO_TAG_LATEST_ARCH} .")
                            } catch (e) {
                                def err = "${ARCH} build failed: ${e}"
                                error(err)
                            }
                            arm_image.push()
                            if (env.BRANCH_NAME == "$PRODUCTION_BRANCH") {
                                arm_image.push("${DOCKER_REPO_TAG_LATEST_ARCH}")
                            }
                        }
                    }
                }
                stage('Build (experimental) manifest') {
                    agent {
                        label 'amd64'
                    }
                    steps {
                        script {
                            sh 'docker manifest create ${DOCKER_REPO_TAG_GIT}_amd64 ${DOCKER_REPO_TAG_GIT}_arm64 ${DOCKER_REPO_TAG_GIT}'
                            if (env.BRANCH_NAME == "$PRODUCTION_BRANCH") {
                                sh 'docker manifest create ${DOCKER_REPO_TAG_LATEST}_amd64 ${DOCKER_REPO_TAG_LATEST}_arm64 ${DOCKER_REPO_TAG_LATEST}'
                            }

                        }
                    }
                }
            }
        }
    }
}
