pipeline {
    agent any

    environment {
        // IMPORTANT: Change this to your actual Docker Hub username!
        DOCKER_REGISTRY = 'arintholia' 
        APP_NAME = 'my-app'
        IMAGE_TAG = "${BUILD_NUMBER}" 
        IMAGE_NAME = "${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                bat "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    // Windows uses %VAR% for environment variables
                    bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                    bat "docker push ${IMAGE_NAME}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'k8s-config', variable: 'KUBECONFIG')]) {
                    // Replaced Linux 'sed' with a Windows PowerShell command to update the image name
                    bat "powershell -Command \"(Get-Content k8s/deployment.yaml) -replace 'IMAGE_PLACEHOLDER', '${IMAGE_NAME}' | Set-Content k8s/deployment.yaml\""
                    bat "kubectl apply -f k8s/deployment.yaml --kubeconfig=\"%KUBECONFIG%\""
                    bat "kubectl apply -f k8s/service.yaml --kubeconfig=\"%KUBECONconfig%\""
                }
            }
        }
    }
}
