pipeline {
    agent any

    environment {
        COMPOSE_PROJECT_NAME = 'devops-appgestiondesprojets'
        TAG = "${BUILD_NUMBER}"
    }

    options {
        timeout(time: 50, unit: 'MINUTES')
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

        stage('Analyse SonarQube') {
            steps {
                dir('backend') {
                    withSonarQubeEnv('sonarqube') {
                        sh 'mvn -B org.sonarsource.scanner.maven:sonar-maven-plugin:5.7.0.6970:sonar -Dsonar.projectKey=appgestion-backend -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.token=$SONAR_AUTH_TOKEN'
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
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

        stage('Push Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        push() { for i in 1 2 3 4 5; do docker push "$1" && return 0; sleep 5; done; return 1; }
                        push samiraffes/appgestion-backend:$TAG
                        push samiraffes/appgestion-frontend:$TAG
                        docker tag samiraffes/appgestion-backend:$TAG samiraffes/appgestion-backend:latest
                        docker tag samiraffes/appgestion-frontend:$TAG samiraffes/appgestion-frontend:latest
                        push samiraffes/appgestion-backend:latest
                        push samiraffes/appgestion-frontend:latest
                    '''
                }
            }
            post {
                always {
                    sh 'docker logout || true'
                }
            }
        }

        stage('Deploy Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f k8s/mysql.yaml
                    kubectl apply -f k8s/prometheus.yaml
                    kubectl apply -f k8s/grafana.yaml
                    sed "s#appgestion-backend:[0-9][0-9]*#appgestion-backend:$TAG#" k8s/backend.yaml | kubectl apply -f -
                    sed "s#appgestion-frontend:[0-9][0-9]*#appgestion-frontend:$TAG#" k8s/frontend.yaml | kubectl apply -f -
                    kubectl rollout status deployment/mysql --timeout=240s
                    kubectl rollout status deployment/backend --timeout=420s
                    kubectl rollout status deployment/frontend --timeout=300s
                    kubectl rollout status deployment/prometheus --timeout=300s
                    kubectl rollout status deployment/grafana --timeout=420s
                '''
            }
        }

        stage('Verification') {
            steps {
                sh '''
                    URL=$(minikube service frontend --url)
                    echo "URL: $URL"
                    for i in $(seq 1 30); do
                        if curl -fs -m 10 "$URL/api/entreprise/all" > /dev/null; then
                            echo "Application OK (Minikube: frontend + backend + MySQL)"
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
            echo 'Pipeline OK : application deployee dans Minikube'
            sh 'docker image prune -f'
        }
        failure {
            echo 'Pipeline en echec : voir le stage rouge'
            sh 'kubectl get pods || true'
            sh 'kubectl logs deploy/backend --tail=40 || true'
        }
    }
}
