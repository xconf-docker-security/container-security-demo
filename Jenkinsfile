pipeline {
    agent any

    environment {
        baseImage = 'openjdk:latest'
        SCANNER_TOKEN = credentials('scanner-token')
    }

    stages {
        stage ("lint dockerfile") {
            agent {
                docker {
                    image 'hadolint/hadolint:latest-debian'
                }
            }
            steps {
                checkout scm
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
        stage('Build App') {
            steps {
                checkout scm
                sh './gradlew build --no-daemon'
            }
        }
        stage('Build Image & Scan') {
            steps {
                checkout scm
                sh 'docker build --build-arg=token=$SCANNER_TOKEN --no-cache .'
            }
        }
    }
}