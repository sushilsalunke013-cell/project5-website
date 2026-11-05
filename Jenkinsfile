pipeline {
    agent any

    environment {
        IMAGE = 'sushilsalunke/project-5-cicd:1.0'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub'
                checkout scm
            }
        }

        stage('Build Image') {
            steps {
                echo 'Building Docker image'

                sh 'docker build -t cloudops-static-site:1.0 .'
            }
        }

        stage('Trivy Scan') {
            steps {
                echo 'Scanning Docker image for HIGH and CRITICAL vulnerabilities'

                sh '''
                    trivy image \
                    --severity HIGH,CRITICAL \
                    --exit-code 1 \
                    cloudops-static-site:1.0
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing image to Docker Hub'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_TOKEN'
                    )
                ]) {
                    sh '''
                        docker tag cloudops-static-site:1.0 $IMAGE

                        echo "$DOCKER_TOKEN" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin

                        docker push $IMAGE
                    '''
                }
            }

            post {
                always {
                    sh 'docker logout || true'
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Image scanned and pushed to Docker Hub!'
        }

        failure {
            echo 'FAILURE: Check the Jenkins Console Output.'
        }

        always {
            echo 'Project 5 CI/CD pipeline execution finished.'
        }
    }
}
