pipeline {
    agent any

    parameters {
        booleanParam(name: 'DEPLOY_TO_K8S', defaultValue: false, description: 'Enable Kubernetes deployment (requires kubeconfig on Jenkins agent)')
    }

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

        stage('Deploy To Kubernetes') {
            when {
                expression { return params.DEPLOY_TO_K8S == true }
            }
            steps {
                bat 'kubectl apply -f deployment.yaml'
                bat 'kubectl apply -f service.yaml'
                bat 'kubectl rollout restart deployment/my-html-deployment'
            }
        }

        stage('Verify') {
            when {
                expression { return params.DEPLOY_TO_K8S == true }
            }
            steps {
                bat 'kubectl get pods'
            }
        }
    }
}