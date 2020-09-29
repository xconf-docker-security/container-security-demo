pipeline {
    agent any

    environment {
        registry = "dockerscannerdemo/container-security-demo"
        registryCredential = 'dockerhub'
        dockerImage = ''
        baseImage = 'postgres:13.0-alpine'
        imageName = "dockerscannerdemo/container-security-demo:$BUILD_NUMBER"
    }

    stages {
        stage ("lint dockerfile") {
            agent {
                docker {
                    image 'hadolint/hadolint:latest-debian'
                }
            }
            steps {
                sh 'hadolint Dockerfile | tee -a hadolint_lint.txt'
            }
            post {
                always {
                    archiveArtifacts 'hadolint_lint.txt'
                }
            }
        }
        stage ("verify signatures") {
            steps {
                sh 'docker trust inspect $baseImage | tee -a signatures.txt'
            }
            post {
                always {
                    archiveArtifacts 'signatures.txt'
                }
            }
        }
        stage('Building image') {
            steps{
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Scan') {
            steps {
                aquaMicroscanner imageName: imageName, notCompliesCmd: 'exit 4', onDisallowed: 'fail', outputFormat: 'html'
            }
        }
        stage('Deploy Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }
    }
}