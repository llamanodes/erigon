def amd_image
def arm_image
def intel_image
def restoreMTime() {
    sh '''
        git restore-mtime
        touch -t $(git show -s --date=format:'%Y%m%d%H%M.%S' --format=%cd HEAD) .git
    '''
}


pipeline {
    agent any
    environment {
        DOCKER_GIT_TAG="$AWS_ECR_URL/erigon:${GIT_COMMIT.substring(0,8)}"
        DOCKER_BUILDKIT=1
    }
    stages {
        stage('build and push') {
            parallel {
                stage('Build and push amd64 image') {
                    agent {
                        label 'amd64_jenkins_agent'
                    }
                    steps {
                        script {
                            DOCKER_GIT_TAG_AMD="$DOCKER_GIT_TAG" + "_amd64"
                            restoreMTime()
                            try {
                                amd_image = docker.build("$DOCKER_GIT_TAG_AMD")
                            } catch (e) {
                                def err = "amd64 build failed: ${e}"
                                error(err)
                            }
                            amd_image.push()
                            amd_image.push('latest')
                            amd_image.push('latest_amd64')
                        }
                    }
                }
                stage('Build and push arm64 image') {
                    agent {
                        label 'arm64_jenkins_agent'
                    }
                    steps {
                        script {
                            DOCKER_GIT_TAG_ARM="$DOCKER_GIT_TAG" + "_arm64"
                            restoreMTime()
                            try {
                                arm_image = docker.build("$DOCKER_GIT_TAG_ARM")
                            } catch (e) {
                                def err = "arm64 build failed: ${e}"
                                error(err)
                            }
                            arm_image.push()
                            arm_image.push('latest_arm64')
                        }
                    }
                }
            }
        }
    }
}
