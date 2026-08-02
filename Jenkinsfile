pipeline {
    agent any

    environment {
        IMAGE_NAME = 'ghcr.io/oj-cyber23/devops-nginx'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Pulling code from GitHub'
                checkout scm
            }
        }

        stage('Deploy with Ansible') {
            steps {
                sh '''
                    pwd
                    ls -R
                    ansible-playbook -i ansible/inventory ansible/nginx.yml
                '''
            }
        }

        stage('Build Container') {
            steps {
                sh '''
                    podman build \
                      -t ${IMAGE_NAME}:${IMAGE_TAG} \
                      -t ${IMAGE_NAME}:latest \
                      .
                '''
            }
        }

        stage('Push to GHCR') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'ghcr-credentials',
                        usernameVariable: 'GHCR_USER',
                        passwordVariable: 'GHCR_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$GHCR_TOKEN" | podman login ghcr.io \
                            --username "$GHCR_USER" \
                            --password-stdin

                        podman push ${IMAGE_NAME}:${IMAGE_TAG}
                        podman push ${IMAGE_NAME}:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    sed "s|IMAGE_PLACEHOLDER|${IMAGE_NAME}:${IMAGE_TAG}|g" \
                        k8s/deployment.yaml > /tmp/deployment.yaml

                    kubectl apply -f /tmp/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    kubectl rollout status deployment/devops-nginx
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    kubectl get pods
                    kubectl get service devops-nginx-service
                '''
            }
        }
    }
}

