
pipeline {
    agent any

    parameters {
        choice(name: 'BUILD_TYPE',
               choices: ['TEST', 'DOCKER'],
               description: 'Select build type')
    }

    environment {
        IMAGE_NAME = 'ewaste-portal'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Parallel Tests') {
            parallel {
                stage('HTML Check') {
                    steps {
                        bat 'if exist index.html (echo HTML file found) else (exit 1)'
                    }
                }

                stage('Dockerfile Check') {
                    steps {
                        bat 'if exist Dockerfile (echo Dockerfile found) else (exit 1)'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            when {
                expression { params.BUILD_TYPE == 'DOCKER' }
            }
            steps {
                bat 'docker build -t ewaste-portal:v1 .'
            }
        }

        stage('Test Application') {
            when {
                expression { params.BUILD_TYPE == 'DOCKER' }
            }
            steps {
                bat 'docker image inspect ewaste-portal:v1'
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'index.html,Dockerfile,Jenkinsfile,deployment.yaml',
                                 fingerprint: true
            }
        }
    }
}
