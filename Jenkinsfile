pipeline {

    agent any

    stages {

        stage('Clone') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t my-html:latest .'
            }
        }

        stage('Load Image To Minikube') {
            steps {
                bat 'minikube image load my-html:latest'
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                bat 'kubectl apply -f deployment.yaml'
                bat 'kubectl apply -f service.yaml'
                bat 'kubectl rollout restart deployment/my-html-deployment'
            }
        }

        stage('Verify') {
            steps {
                bat 'kubectl get pods'
            }
        }
    }
}