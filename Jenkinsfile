pipeline {
    agent none
    triggers {
        cron('@weekly')
        pollSCM('*/15 * * * *')
    }
    environment {
        DOCKER_REPO = "stevbev/docker-registry-ui"
        TAG = ""
        VERSION_FILE = "version.go"
    }
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Clean workspace') {
            parallel {
                stage('amd64') {
                    agent {
                        label 'amd64'
                    }
                    steps {
                        cleanWs()
                    }
                }
                stage('arm') {
                    agent {
                        label 'arm'
                    }
                    steps {
                        cleanWs()
                    }
                }
                stage('arm64') {
                    agent {
                        label 'arm64'
                    }
                    steps {
                        cleanWs()
                    }
                }
            }
        }
        stage('Docker Images') {
            parallel {
                stage('amd64') {
                    agent {
                        label 'amd64'
                    }
                    steps {
                        script {
                            TAG = sh(returnStdout: true, script: "grep -i 'version' ${VERSION_FILE} | sed \"s/[^0-9.]//g\"").trim()
                            docker.withRegistry('', 'dockerHub') {
                                def image = docker.build("${DOCKER_REPO}:${TAG}-amd64", "-f Dockerfile .")
                                image.push()
                            }
                        }
                    }
                }
                stage('arm') {
                    agent {
                        label 'arm'
                    }
                    steps {
                        script {
                            TAG = sh(returnStdout: true, script: "grep -i 'version' ${VERSION_FILE} | sed \"s/[^0-9.]//g\"").trim()
                            docker.withRegistry('', 'dockerHub') {
                                def image = docker.build("${DOCKER_REPO}:${TAG}-arm", "-f Dockerfile .")
                                image.push()
                            }
                        }
                    }
                }
                stage('arm64') {
                    agent {
                        label 'arm64'
                    }
                    steps {
                        script {
                            TAG = sh(returnStdout: true, script: "grep -i 'version' ${VERSION_FILE} | sed \"s/[^0-9.]//g\"").trim()
                            docker.withRegistry('', 'dockerHub') {
                                def image = docker.build("${DOCKER_REPO}:${TAG}-arm64", "-f Dockerfile .")
                                image.push()
                            }
                        }
                    }
                }
            }
        }
        stage('Docker Manifest') {
            agent {
                label 'amd64'
            }
            steps {
                script {
                    docker.withRegistry('', 'dockerHub') {
                        TAG = sh(returnStdout: true, script: "grep -i 'version' ${VERSION_FILE} | sed \"s/[^0-9.]//g\"").trim()
                        sh(script: "docker manifest create ${DOCKER_REPO}:${TAG} ${DOCKER_REPO}:${TAG}-amd64 ${DOCKER_REPO}:${TAG}-arm ${DOCKER_REPO}:${TAG}-arm64")
                        sh(script: "docker manifest inspect ${DOCKER_REPO}:${TAG}")
                        sh(script: "docker manifest push -p ${DOCKER_REPO}:${TAG}")
                        sh(script: "docker manifest create ${DOCKER_REPO}:latest ${DOCKER_REPO}:${TAG}-amd64 ${DOCKER_REPO}:${TAG}-arm ${DOCKER_REPO}:${TAG}-arm64")
                        sh(script: "docker manifest inspect ${DOCKER_REPO}:latest")
                        sh(script: "docker manifest push -p ${DOCKER_REPO}:latest")
                    }
                }
            }
        }
    }
}
