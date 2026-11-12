
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

        stage('Check Project') {
            steps {
                echo 'Checking Maven project...'
                sh 'pwd'
                sh 'ls -la'
                sh 'find . -name pom.xml'
            }
        }

        stage('Clean') {
            steps {
                echo 'Cleaning project...'
                sh 'mvn clean'
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging project...'
                sh 'mvn package -DskipTests'
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
