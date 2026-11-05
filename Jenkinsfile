pipeline {
    agent any

    environment {
        IMAGE = 'sushilsalunke/project-5-cicd:1.0'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t cloudops-static-site:1.0 .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image --severity HIGH,CRITICAL --exit-code 1 cloudops-static-site:1.0'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh '''
                        docker tag cloudops-static-site:1.0 $IMAGE

                        echo "$DOCKER_TOKEN" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker push $IMAGE
                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Image scanned and pushed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}
