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
                dir('gestion-projets') {
                    sh 'mvn clean'
                }
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging project...'
                dir('gestion-projets') {
                    sh 'mvn package -DskipTests'
                }
            }
        }

    }

    post {
        success {
            echo 'Build successful !'
        }
        failure {
            echo 'Build failed !'
        }
    }
}
