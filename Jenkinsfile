pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"  // Change this to your AWS region
        AWS_ACCOUNT_ID = "851725270304"  // Replace with your AWS Account ID
        REPO_NAME = "adservice"  // Replace with your ECR repository name
        IMAGE_TAG = "latest"
        ECR_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}"
    }

    stages {
        stage('Login to AWS ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_URL}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dir('src') {
                        sh "docker build -t ${REPO_NAME}:${IMAGE_TAG} ."
                    }
                }
            }
        }

        stage('Push Docker Image to AWS ECR') {
            steps {
                script {
                    sh """
                        docker tag ${REPO_NAME}:${IMAGE_TAG} ${ECR_URL}:${IMAGE_TAG}
                        docker push ${ECR_URL}:${IMAGE_TAG}
                    """
                }
            }
        }
    }
}
