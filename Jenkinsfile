
pipeline {
    agent any

    environment {
        IMAGE = "sushilsalunke/project-5-cicd:1.0-${BUILD_NUMBER}"
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
                echo 'Scanning Docker image'

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

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying application to Kubernetes'

                withCredentials([
                    string(
                        credentialsId: 'k8s-token',
                        variable: 'K8S_TOKEN'
                    )
                ]) {
                    sh '''
                        kubectl \
                        --server=https://172.31.43.38:6443 \
                        --certificate-authority=/tmp/ca.crt \
                        --token="$K8S_TOKEN" \
                        set image deployment/project5-website \
                        project5-website=$IMAGE

                        kubectl \
                        --server=https://172.31.43.38:6443 \
                        --certificate-authority=/tmp/ca.crt \
                        --token="$K8S_TOKEN" \
                        rollout status deployment/project5-website \
                        --timeout=180s
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Application deployed to Kubernetes!'
        }

        failure {
            echo 'FAILURE: Check Jenkins Console Output.'
        }

        always {
            echo 'Project 5 CI/CD pipeline finished.'
        }
    }
}
