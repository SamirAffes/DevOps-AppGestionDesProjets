pipeline {
    agent any

    environment {
        // Même nom que ta stack lancée à la main : Jenkins reprend les mêmes
        // conteneurs et le même volume MySQL (pas de conflit de noms).
        COMPOSE_PROJECT_NAME = 'devops-appgestiondesprojets'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }

    stages {
        stage('Pull') {
            steps {
                checkout scm
            }
        }

        stage('Test backend') {
            steps {
                dir('backend') {
                    sh 'mvn -B clean test'
                }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'backend/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package backend') {
            steps {
                dir('backend') {
                    sh 'mvn -B package -DskipTests'
                }
            }
        }

        stage('Build images Docker') {
            steps {
                sh 'docker compose build'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker compose up -d'
            }
        }

        stage('Verification') {
            steps {
                sh '''
                    for i in $(seq 1 30); do
                        if curl -fs http://localhost:8081/entreprise/all > /dev/null \
                           && curl -fs http://localhost:4200/api/entreprise/all > /dev/null; then
                            echo "Application OK (backend + frontend via Nginx)"
                            exit 0
                        fi
                        echo "En attente de l'application ($i/30)..."
                        sleep 5
                    done
                    echo "L'application ne repond pas"
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline OK : application disponible sur le port 4200'
            sh 'docker image prune -f'
        }
        failure {
            echo 'Pipeline en echec : voir le stage rouge'
            sh 'docker compose ps || true'
            sh 'docker compose logs --tail=40 backend || true'
        }
    }
}
