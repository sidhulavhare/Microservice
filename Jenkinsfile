pipeline {
    agent any

    environment {
        AWS_REGION     = "ap-south-1"                       // update with your AWS region
        AWS_ACCOUNT_ID = "427601800855"                    // update with your AWS account ID
        REPO_NAME      = "my-app-repo"                     // your ECR repo name
        IMAGE_TAG      = "latest"                          // or use BUILD_NUMBER for unique tag
                                                        // folder path containing your Dockerfile
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/sidhulavhare/Microservice.git'
            }
        }

        stage('Create ECR Repository') {
            steps {
                script {
                    sh """
                    aws ecr describe-repositories --repository-names ${REPO_NAME} --region ${AWS_REGION} || \
                    aws ecr create-repository --repository-name ${REPO_NAME} --region ${AWS_REGION}
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t ${REPO_NAME}:${IMAGE_TAG} .
                    """
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh """
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    docker tag ${REPO_NAME}:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}:${IMAGE_TAG}
                    docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
    }
}
