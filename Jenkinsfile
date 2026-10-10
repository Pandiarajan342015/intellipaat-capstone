
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
    }
}
