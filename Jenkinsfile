pipeline {
    agent any

    environment {
        // We will update this username later!
        DOCKER_REGISTRY = 'your_dockerhub_username' 
        APP_NAME = 'my-app'
        IMAGE_TAG = "${BUILD_NUMBER}" 
        IMAGE_NAME = "${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    sh "docker push ${IMAGE_NAME}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'k8s-config', variable: 'KUBECONFIG')]) {
                    sh "sed -i 's|IMAGE_PLACEHOLDER|${IMAGE_NAME}|g' k8s/deployment.yaml"
                    sh "kubectl apply -f k8s/deployment.yaml --kubeconfig=\$KUBECONFIG"
                    sh "kubectl apply -f k8s/service.yaml --kubeconfig=\$KUBECONFIG"
                }
            }
        }
    }
}
