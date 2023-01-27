def image
def amd_image
def arm_image
def intel_image


pipeline {
    agent any
    environment {
        DOCKER_GIT_TAG="$AWS_ECR_URL/$JOB_BASE_NAME:${$GIT_COMMIT.substring(0,8)}"
        DOCKER_LATEST_TAG="$AWS_ECR_URL/$JOB_BASE_NAME:latest"
        DOCKER_BUILDKIT=1
    }
    stages {
        stage('build') {
            parallel {
                stage('Build amd64 image') {
                    agent {label 'amd64_jenkins_agent'}
                    steps {
                        script {
                            DOCKER_GIT_TAG_AMD="$DOCKER_GIT_TAG" + "_amd64"
                            amd_image = docker.build("$DOCKER_GIT_TAG_AMD")
                        }
                    }
                }
                stage('Build arm64 image') {
                    agent {label 'arm64_jenkins_agent'}
                    steps {
                        script {
                            DOCKER_GIT_TAG_ARM="$DOCKER_GIT_TAG" + "_arm64"
                            arm_image = docker.build("$DOCKER_GIT_TAG_ARM")
                        }
                    }
                }
                stage('Build intel image') {
                    agent {label 'intel_jenkins_agent'}
                    steps {
                        script {
                            DOCKER_GIT_TAG_INTEL="$DOCKER_GIT_TAG" + "_intel_sky_lake"
                            intel_image = docker.build("$DOCKER_GIT_TAG_INTEL")
                        }
                    }
                }
            }
        }
        stage('push') {
            parallel {
                stage('push amd64 image') {
                    agent {label 'amd64_jenkins_agent'}
                    steps {
                        script {
                            amd_image.push()
                            amd_image.push('latest_amd3')
                        }
                    }
                }
                stage('push arm64 image') {
                    agent {label 'arm64_jenkins_agent'}
                    steps {
                        script {
                            arm_image.push()
                            arm_image.push('latest_graviton2')
                        }
                    }
                }
                stage('push intel image') {
                    agent {label 'intel_jenkins_agent'}
                    steps {
                        script {
                            intel_image.push()
                            intel_image.push('latest')
                        }
                    }
                }
            }
        }  
    }
}