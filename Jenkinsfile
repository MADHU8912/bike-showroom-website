pipeline {
    agent any

    environment {
        IMAGE_NAME = "bike-showroom"
        DOCKERHUB_USER = "nikhilabba12"
        IMAGE_TAG = "${BUILD_NUMBER}"

        ACR_NAME = "myacrname"
        ACR_LOGIN_SERVER = "myacrname.azurecr.io"
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
                docker build -t %nikhilabba12%/%IMAGE_NAME%:%IMAGE_TAG% .
                docker tag %nikhilabba12%/%IMAGE_NAME%:%IMAGE_TAG% %DOCKERHUB_USER%/%IMAGE_NAME%:latest
                """
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

        stage('Push To Docker Hub') {
            steps {
                bat """
                docker push %nikhilabba12%/%IMAGE_NAME%:%IMAGE_TAG%
                docker push %nikhilabba12%/%IMAGE_NAME%:latest
                """
            }
        }

        stage('Tag Image For Azure ACR') {
            steps {
                bat """
                docker tag %nikhilabba12%/%IMAGE_NAME%:%IMAGE_TAG% %ACR_LOGIN_SERVER%/%IMAGE_NAME%:%IMAGE_TAG%
                docker tag %nikhilabba12%/%IMAGE_NAME%:%IMAGE_TAG% %ACR_LOGIN_SERVER%/%IMAGE_NAME%:latest
                """
            }
        }

        stage('Azure Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'acr-creds',
                        usernameVariable: 'ACR_USER',
                        passwordVariable: 'ACR_PASS'
                    )
                ]) {
                    bat """
                    echo %ACR_PASS% | docker login %ACR_LOGIN_SERVER% -u %ACR_USER% --password-stdin
                    """
                }
            }
        }

        stage('Push To Azure ACR') {
            steps {
                bat """
                docker push %ACR_LOGIN_SERVER%/%IMAGE_NAME%:%IMAGE_TAG%
                docker push %ACR_LOGIN_SERVER%/%IMAGE_NAME%:latest
                """
            }
        }

        stage('Show Images') {
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