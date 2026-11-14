pipeline {
    agent any

    stages {

        stage('Pull') {
            steps {
                echo 'Pulling from GitHub...'
                git branch: 'main',
                    url: 'https://github.com/SamirAffes/DevOps-AppGestionDesProjets.git'
            }
        }

        stage('Clean') {
            steps {
                echo 'Cleaning project...'
                dir('backend') {
                    sh 'mvn clean'
                }
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging project...'
                dir('backend') {
                    sh 'mvn package -DskipTests'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images...'
                sh 'docker compose build'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying stack...'
                sh 'docker compose up -d'
            }
        }

    }

    post {
        success {
            echo 'Deployment successful !'
        }
        failure {
            echo 'Deployment failed !'
        }
    }
}
