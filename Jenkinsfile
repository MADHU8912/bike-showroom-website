pipeline {
    agent any

    environment {
        IMAGE_NAME = "bike-showroom"
        IMAGE_TAG  = "${BUILD_NUMBER}"

        DOCKERHUB_USER = "nikhilabba12"
        ACR_NAME = "youracrname"
        ACR_LOGIN_SERVER = "youracrname.azurecr.io"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Files') {
            steps {
                bat 'dir'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat """
                docker build -t %IMAGE_NAME%:%IMAGE_TAG% .
                """
            }
        }

        stage('Show Images') {
            steps {
                bat 'docker images'
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat '''
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    '''
                }
            }
        }

        stage('Tag For Docker Hub') {
            steps {
                bat """
                docker tag %IMAGE_NAME%:%IMAGE_TAG% %DOCKERHUB_USER%/%IMAGE_NAME%:latest
                docker tag %IMAGE_NAME%:%IMAGE_TAG% %DOCKERHUB_USER%/%IMAGE_NAME%:%IMAGE_TAG%
                """
            }
        }

        stage('Push To Docker Hub') {
            steps {
                bat """
                docker push %DOCKERHUB_USER%/%IMAGE_NAME%:latest
                docker push %DOCKERHUB_USER%/%IMAGE_NAME%:%IMAGE_TAG%
                """
            }
        }

        stage('Tag Image For Azure ACR') {
            steps {
                bat """
                docker tag %IMAGE_NAME%:%IMAGE_TAG% %ACR_LOGIN_SERVER%/%IMAGE_NAME%:%IMAGE_TAG%
                """
            }
        }

        stage('Azure Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'azure-creds',
                        usernameVariable: 'AZURE_USER',
                        passwordVariable: 'AZURE_PASS'
                    )
                ]) {
                    bat """
                    az login -u %AZURE_USER% -p %AZURE_PASS%
                    az acr login --name %ACR_NAME%
                    """
                }
            }
        }

        stage('Push To Azure ACR') {
            steps {
                bat """
                docker push %ACR_LOGIN_SERVER%/%IMAGE_NAME%:%IMAGE_TAG%
                """
            }
        }

        stage('Show Images After Push') {
            steps {
                bat 'docker images'
            }
        }
    }

    post {
        always {
            bat 'docker logout'
        }

        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
}