
pipeline {
    agent any

    parameters {
        choice(name: 'BUILD_TYPE',
               choices: ['TEST', 'DOCKER'],
               description: 'Choose the pipeline operation')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Parallel Tests') {
            parallel {
                stage('HTML File Check') {
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
                bat 'docker build -t ewaste-portal:latest .'
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'index.html,Dockerfile,Jenkinsfile', fingerprint: true
            }
        }
    }
}
