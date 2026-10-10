pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'docker2616/intellipaat-website'
        K8S_API      = 'https://172.31.35.218:6443'
        K8S_NAMESPACE = 'default'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Tools') {
            steps {
                sh '''
                    git --version
                    docker --version
                    kubectl version --client
                '''
            }
        }

        stage('Verify Website') {
            steps {
                sh '''
                    test -f index.html
                    test -f Dockerfile
                    echo "Website files verified"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_TOKEN'
                )]) {
                    sh '''
                        set +x
                        echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([string(
                    credentialsId: 'k8s-deployer-token',
                    variable: 'K8S_TOKEN'
                )]) {
                    sh '''
                        set +x
                        set -eu

                        KUBECONFIG_FILE="$(mktemp)"
                        trap 'rm -f "$KUBECONFIG_FILE"' EXIT
                        chmod 600 "$KUBECONFIG_FILE"
                        export KUBECONFIG="$KUBECONFIG_FILE"

                        kubectl config set-cluster capstone \
                          --server="$K8S_API" \
                          --insecure-skip-tls-verify=true \
                          --kubeconfig="$KUBECONFIG_FILE"

                        kubectl config set-credentials jenkins-deployer \
                          --token="$K8S_TOKEN" \
                          --kubeconfig="$KUBECONFIG_FILE"

                        kubectl config set-context capstone \
                          --cluster=capstone \
                          --user=jenkins-deployer \
                          --namespace="$K8S_NAMESPACE" \
                          --kubeconfig="$KUBECONFIG_FILE"

                        kubectl --kubeconfig="$KUBECONFIG_FILE" \
                          set image deployment/intellipaat-website \
                          website=${DOCKER_IMAGE}:${BUILD_NUMBER}

                        kubectl --kubeconfig="$KUBECONFIG_FILE" \
                          rollout status deployment/intellipaat-website \
                          --timeout=180s

                        kubectl --kubeconfig="$KUBECONFIG_FILE" \
                          get deployment intellipaat-website
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Build, push, and Kubernetes deployment completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Review the stage logs before retrying.'
        }
    }
}
