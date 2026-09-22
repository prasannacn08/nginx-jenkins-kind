pipeline {

    agent any

    environment {

        DOCKER_IMAGE = 'YOUR_DOCKERHUB_USERNAME/nginx-app'
        IMAGE_TAG = 'latest'

        DEPLOYMENT_NAME = 'nginx-deployment'
        CONTAINER_NAME = 'nginx'
    }

    stages {

        stage('Checkout') {

            steps {

                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/YOUR_USERNAME/nginx-jenkins-kind.git'
            }
        }


        stage('Test') {

            steps {

                sh '''
                    test -f Dockerfile
                    test -f index.html
                    test -f deployment.yaml
                    test -f service.yaml

                    echo "Required files are present"
                '''
            }
        }


        stage('Docker Build') {

            steps {

                sh '''
                    docker build \
                    -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                '''
            }
        }


        stage('Docker Login') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | \
                        docker login \
                        --username "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }


        stage('Push to Docker Hub') {

            steps {

                sh '''
                    docker push \
                    ${DOCKER_IMAGE}:${IMAGE_TAG}
                '''
            }
        }


        stage('Kubernetes Deploy') {

            steps {

                sh '''
                    kubectl apply -f deployment.yaml

                    kubectl apply -f service.yaml

                    kubectl set image \
                    deployment/${DEPLOYMENT_NAME} \
                    ${CONTAINER_NAME}=${DOCKER_IMAGE}:${IMAGE_TAG}
                '''
            }
        }


        stage('Deployment Verification') {

            steps {

                sh '''
                    kubectl rollout status \
                    deployment/${DEPLOYMENT_NAME} \
                    --timeout=120s

                    echo "===== NODES ====="

                    kubectl get nodes

                    echo "===== PODS ====="

                    kubectl get pods

                    echo "===== SERVICE ====="

                    kubectl get svc
                '''
            }
        }
    }
}
