pipeline {

    agent any

    stages {

        stage('Clone') {
            steps {
                git 'https://github.com/MADHU871/bike-showroom-website.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t bike-showroom .'
            }
        }

        stage('Push Docker Image') {
            steps {
                bat 'docker tag bike-showroom nikhilabba12/bike-showroom:latest'
                bat 'docker push nikhilabba12/bike-showroom:latest'
            }
        }
    }
}