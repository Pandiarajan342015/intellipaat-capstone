
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Source code is available from GitHub.'
                sh 'pwd'
                sh 'ls -la'
            }
        }

        stage('Verify Tools') {
            steps {
                sh 'java -version'
                sh 'git --version'
            }
        }

        stage('Verify Website') {
            steps {
                sh 'test -f index.html'
                echo 'Website index.html found successfully.'
            }
        }
    
            stage('Build Docker Image') {
            steps {
                sh 'docker build -t intellipaat-website:${BUILD_NUMBER} -t intellipaat-website:latest .'
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
                        echo "$DOCKERHUB_TOKEN" | docker login \
                            --username "$DOCKERHUB_USER" \
                            --password-stdin
                        docker tag intellipaat-website:${BUILD_NUMBER} \
                            "$DOCKERHUB_USER/intellipaat-website:${BUILD_NUMBER}"
                        docker tag intellipaat-website:${BUILD_NUMBER} \
                            "$DOCKERHUB_USER/intellipaat-website:latest"
                        docker push "$DOCKERHUB_USER/intellipaat-website:${BUILD_NUMBER}"
                        docker push "$DOCKERHUB_USER/intellipaat-website:latest"
                        docker logout
                    '''
                }
            }
        }
  
    
    }
    
}
