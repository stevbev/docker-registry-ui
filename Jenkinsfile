pipeline {
    agent none
    environment {
        DOCKER_REPO = "stevbev/docker-registry-ui"
        TAG = ""
        VERSION_FILE = "version.go"
    }
    options {
        skipStagesAfterUnstable()
    }
    stages {
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
            }
        }
        stage('Docker Manifest') {
            agent {
                label 'arm'
            }
            steps {
                script {
                    docker.withRegistry('', 'dockerHub') {
                        TAG = sh(returnStdout: true, script: "grep -i 'version' ${VERSION_FILE} | sed \"s/[^0-9.]//g\"").trim()
                        sh(script: "docker manifest create ${DOCKER_REPO}:${TAG} ${DOCKER_REPO}:${TAG}-amd64 ${DOCKER_REPO}:${TAG}-arm")
                        sh(script: "docker manifest inspect ${DOCKER_REPO}:${TAG}")
                        sh(script: "docker manifest push -p ${DOCKER_REPO}:${TAG}")
                        sh(script: "docker manifest create ${DOCKER_REPO}:latest ${DOCKER_REPO}:${TAG}-amd64 ${DOCKER_REPO}:${TAG}-arm")
                        sh(script: "docker manifest inspect ${DOCKER_REPO}:latest")
                        sh(script: "docker manifest push -p ${DOCKER_REPO}:latest")
                    }
                }
            }
        }
    }
}
