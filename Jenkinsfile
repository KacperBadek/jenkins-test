pipeline {
    agent any

    environment {
        NODE_ENV = 'test'
        APP_DIR = 'app'
        IMAGE_NAME = 'hasacz325/jenkins-testing'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
    }

    tools {
        nodejs 'NodeJS'
    }

    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Install dependencies') {
            steps {
                dir("${APP_DIR}") {
                    sh 'npm ci'
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                dir("${APP_DIR}") {
                    sh 'npm test'
                }
            }
        }

        stage('Build Application') {
            steps {
                dir("${APP_DIR}") {
                    sh 'npm run build || echo "Brak procesu build, pomijam"'
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_NAME}:latest", "${APP_DIR}")
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Code Coverage Report') {
                    steps {
                        dir("${APP_DIR}") {
                            sh 'npm run test:coverage'
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: true,
                                keepAll: true,
                                reportDir: 'coverage/lcov-report',
                                reportFiles: 'index.html',
                                reportName: 'Code Coverage'
                            ])
                        }
                    }
                }
    }

    post {
        failure {
            echo 'Pipeline failed.'
        }
        success {
            echo 'Pipeline completed successfully.'
        }
    }
}
