pipeline {
    agent any
    options {
        timeout(time: 45, unit: 'MINUTES')
        ansiColor('xterm')
    }

    parameters {
        booleanParam(name: 'DEPLOY_TO_K8S', defaultValue: false, description: 'Enable Kubernetes deployment (requires kubeconfig credential)')
        booleanParam(name: 'PUSH_TO_REGISTRY', defaultValue: false, description: 'Push image to container registry')
        string(name: 'IMAGE_NAME', defaultValue: 'my-html', description: 'Docker image name (without tag)')
        string(name: 'REGISTRY', defaultValue: '', description: 'Container registry (e.g. registry.example.com/myorg)')
        string(name: 'DOCKER_CREDENTIALS_ID', defaultValue: '', description: 'Jenkins credentials id for docker registry (username/password)')
        string(name: 'KUBECONFIG_CRED_ID', defaultValue: '', description: 'Jenkins credentials id for kubeconfig (file)')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    def cmd = "docker build -t ${params.IMAGE_NAME}:latest ."
                    if (isUnix()) {
                        sh cmd
                    } else {
                        bat cmd
                    }
                }
            }
        }

        stage('Push to Registry') {
            when {
                expression { return params.PUSH_TO_REGISTRY && params.REGISTRY?.trim() }
            }
            steps {
                script {
                    def fullImage = "${params.REGISTRY}/${params.IMAGE_NAME}:latest"
                    withCredentials([usernamePassword(credentialsId: params.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        if (isUnix()) {
                            sh "echo $DOCKER_PASS | docker login ${params.REGISTRY} -u $DOCKER_USER --password-stdin"
                            sh "docker tag ${params.IMAGE_NAME}:latest ${fullImage}"
                            sh "docker push ${fullImage}"
                        } else {
                            bat "echo %DOCKER_PASS% | docker login ${params.REGISTRY} -u %DOCKER_USER% --password-stdin"
                            bat "docker tag ${params.IMAGE_NAME}:latest ${fullImage}"
                            bat "docker push ${fullImage}"
                        }
                    }
                }
            }
        }

        stage('Load to Minikube') {
            when {
                expression { return !params.PUSH_TO_REGISTRY }
            }
            steps {
                script {
                    def cmd = "minikube image load ${params.IMAGE_NAME}:latest --overwrite"
                    if (isUnix()) { sh cmd } else { bat cmd }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when { expression { return params.DEPLOY_TO_K8S } }
            steps {
                script {
                    withCredentials([file(credentialsId: params.KUBECONFIG_CRED_ID, variable: 'KUBECONFIG_FILE')]) {
                        if (isUnix()) {
                            sh "kubectl --kubeconfig=$KUBECONFIG_FILE apply -f deployment.yaml"
                            sh "kubectl --kubeconfig=$KUBECONFIG_FILE apply -f service.yaml"
                            sh "kubectl --kubeconfig=$KUBECONFIG_FILE rollout restart deployment/my-html-deployment || true"
                        } else {
                            bat "kubectl --kubeconfig=%KUBECONFIG_FILE% apply -f deployment.yaml"
                            bat "kubectl --kubeconfig=%KUBECONFIG_FILE% apply -f service.yaml"
                            bat "kubectl --kubeconfig=%KUBECONFIG_FILE% rollout restart deployment/my-html-deployment || exit 0"
                        }
                    }
                }
            }
        }

        stage('Verify') {
            when { expression { return params.DEPLOY_TO_K8S } }
            steps {
                script {
                    if (isUnix()) {
                        sh "kubectl --kubeconfig=$KUBECONFIG_FILE get pods -o wide"
                    } else {
                        bat "kubectl --kubeconfig=%KUBECONFIG_FILE% get pods -o wide"
                    }
                }
            }
        }
    }

    post {
        success { echo 'Pipeline completed successfully.' }
        failure { echo 'Pipeline failed — check the logs.' }
    }
}