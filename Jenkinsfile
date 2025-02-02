pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"  // Change this to your AWS region
        AWS_ACCOUNT_ID = "851725270304"  // Replace with your AWS Account ID
        REPO_NAME = "adservice"  // Replace with your ECR repository name
        IMAGE_TAG = "latest"
        ECR_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}"
    }
        stage('Login to AWS ECR') {
            steps {
                script {
                    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 851725270304.dkr.ecr.us-east-1.amazonaws.com
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dir('src'){
                       docker build -t adservice .
                    }
                
                }
            }
        }

        stage('Push Docker Image to AWS ECR') {
            steps {
                script {
                    sh "docker tag adservice:latest 851725270304.dkr.ecr.us-east-1.amazonaws.com/adservice:latest"
                    sh "docker push 851725270304.dkr.ecr.us-east-1.amazonaws.com/adservice:latest"
                }
            }
        }

       
    }
}
