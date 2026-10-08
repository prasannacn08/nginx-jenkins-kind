
pipeline {

    agent any

    environment {

        DOCKER_IMAGE = '95prasanna/nginx-app'
        IMAGE_TAG = "${BUILD_NUMBER}"

        DEPLOYMENT_NAME = 'nginx-production'
        CONTAINER_NAME = 'nginx'

        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {

        stage('Checkout') {

            steps {

                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/prasannacn08/nginx-jenkins-kind.git'
            }
        }


        stage('Test') {

            steps {

                sh '''
                    echo "===== Checking Application Files ====="

                    test -f Dockerfile
                    test -f index.html

                    echo "===== Checking Kubernetes Files ====="

                    test -f k8s/namespace.yaml
                    test -f k8s/configmap.yaml
                    test -f k8s/secret.yaml
                    test -f k8s/pv.yaml
                    test -f k8s/pvc.yaml
                    test -f k8s/deployment.yaml
                    test -f k8s/service.yaml
                    test -f k8s/hpa.yaml
                    test -f k8s/ingress.yaml

                    echo "All required files are present."
                '''
            }
        }


        stage('Docker Build') {

            steps {

                sh '''
                    echo "===== Building Docker Image ====="

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
                    echo "===== Pushing Image to Docker Hub ====="

                    docker push \
                    ${DOCKER_IMAGE}:${IMAGE_TAG}

                    docker logout
                '''
            }
        }


        stage('Kubernetes Deploy') {

            steps {

                sh '''
                    echo "===== Kubernetes Cluster ====="

                    kubectl get nodes

                    echo "===== Creating Namespace ====="

                    kubectl apply \
                    -f k8s/namespace.yaml

                    echo "===== Applying ConfigMap ====="

                    kubectl apply \
                    -f k8s/configmap.yaml

                    echo "===== Applying Secret ====="

                    kubectl apply \
                    -f k8s/secret.yaml

                    echo "===== Applying Persistent Volume ====="

                    kubectl apply \
                    -f k8s/pv.yaml

                    echo "===== Applying Persistent Volume Claim ====="

                    kubectl apply \
                    -f k8s/pvc.yaml

                    echo "===== Applying Deployment ====="

                    kubectl apply \
                    -f k8s/deployment.yaml

                    echo "===== Applying Service ====="

                    kubectl apply \
                    -f k8s/service.yaml

                    echo "===== Applying HPA ====="

                    kubectl apply \
                    -f k8s/hpa.yaml

                    echo "===== Applying Ingress ====="

                    kubectl apply \
                    -f k8s/ingress.yaml

                    echo "===== Updating Docker Image ====="

                    kubectl -n production set image \
                    deployment/${DEPLOYMENT_NAME} \
                    ${CONTAINER_NAME}=${DOCKER_IMAGE}:${IMAGE_TAG}
                '''
            }
        }


        stage('Deployment Verification') {

            steps {

                sh '''
                    echo "===== Waiting for Rollout ====="

                    kubectl rollout status \
                    deployment/${DEPLOYMENT_NAME} \
                    -n production \
                    --timeout=120s


                    echo ""
                    echo "===== NODES ====="

                    kubectl get nodes


                    echo ""
                    echo "===== PODS ====="

                    kubectl get pods \
                    -n production \
                    -o wide


                    echo ""
                    echo "===== DEPLOYMENT ====="

                    kubectl get deployment \
                    -n production


                    echo ""
                    echo "===== SERVICE ====="

                    kubectl get svc \
                    -n production


                    echo ""
                    echo "===== HPA ====="

                    kubectl get hpa \
                    -n production


                    echo ""
                    echo "===== PVC ====="

                    kubectl get pvc \
                    -n production


                    echo ""
                    echo "===== INGRESS ====="

                    kubectl get ingress \
                    -n production


                    echo ""
                    echo "===== DEPLOYED IMAGE ====="

                    kubectl get deployment \
                    ${DEPLOYMENT_NAME} \
                    -n production \
                    -o jsonpath='{.spec.template.spec.containers[0].image}'

                    echo ""
                '''    
            }
        }
    }
}
