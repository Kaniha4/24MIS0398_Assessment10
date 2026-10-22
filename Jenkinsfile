
pipeline {
    agent any

    parameters {
        choice(name: 'BUILD_TYPE',
               choices: ['TEST', 'DOCKER'],
               description: 'Select build type')
    }

    environment {
        DOCKER_EXE = 'C:\\Users\\91822\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        IMAGE_NAME = 'kaniha4/ewaste-portal'
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
                bat '"%DOCKER_EXE%" build -t ewaste-portal:v1 .'
            }
        }

        stage('Test Application') {
            when {
                expression { params.BUILD_TYPE == 'DOCKER' }
            }
            steps {
                bat '"%DOCKER_EXE%" image inspect ewaste-portal:v1'
            }
        }

        stage('Push to Docker Hub') {
            when {
                expression { params.BUILD_TYPE == 'DOCKER' }
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    powershell '''
                        $docker = $env:DOCKER_EXE
                        $env:DOCKER_TOKEN | & $docker login --username $env:DOCKER_USER --password-stdin
                        if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }

                        & $docker tag ewaste-portal:v1 "$env:IMAGE_NAME:v1"
                        if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }

                        & $docker push "$env:IMAGE_NAME:v1"
                        if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
                    '''
                }
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
