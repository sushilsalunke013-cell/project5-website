pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out Project 5 source code'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'
                sh 'docker build -t cloudops-static-site:1.0 .'
            }
        }

        stage('Trivy Security Scan') {
            steps {
                echo 'Scanning Docker image for vulnerabilities'
                sh '''
                    trivy image \
                    --severity HIGH,CRITICAL \
                    --exit-code 1 \
                    cloudops-static-site:1.0
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed: image passed the security gate.'
        }
        failure {
            echo 'Pipeline failed. Check the build or Trivy scan output.'
        }
    }
}
