pipeline {
    agent any

    environment {
        BACKEND_IMAGE = 'arxisthingy/tp1-backend:latest'
        FRONTEND_IMAGE = 'arxisthingy/tp1-frontend:latest'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Checkout berhasil!'
            }
        }

        stage('Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    // Login ke Docker Hub
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    
                    // Build & Push Backend
                    sh "docker build -t ${BACKEND_IMAGE} ./backend"
                    sh "docker push ${BACKEND_IMAGE}"
                    
                    // Build & Push Frontend
                    sh "docker build -t ${FRONTEND_IMAGE} ./frontend"
                    sh "docker push ${FRONTEND_IMAGE}"
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-file', variable: 'KUBECONFIG')]) {
                    // Deploy K8s Deployment & Service
                    sh 'kubectl apply -f k8s.yaml'
                    // Deploy K8s Ingress
                    sh 'kubectl apply -f ingress.yaml'
                    
                    sh 'kubectl rollout restart deployment backend-deployment'
                    sh 'kubectl rollout restart deployment frontend-deployment'
                }
            }
        }
    }
}