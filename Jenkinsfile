pipeline {
    agent any

    environment {
        AWS_REGION     = "ap-south-1"
        AWS_ACCOUNT_ID = "427601800855"
        REPO_NAME      = "frontend"
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }


    stages {
        stage('Checkout Source Code') {
            steps {
                echo 'Checking out source code...'
                // Replace the URL and branch with your repo details
                git branch: 'adservice', url: 'https://github.com/sidhulavhare/Microservice.git'
            }
        }

        stage('AWS Login & Create ECR Repo') {
            steps {
                withAWS(credentials: 'aws-jenkins-credentials', region: "${AWS_REGION}") {
                    sh '''
                    echo "Logging into AWS ECR..."
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

                    echo "Check or create ECR repository..."
                    aws ecr describe-repositories --repository-names $REPO_NAME --region $AWS_REGION || \
                    aws ecr create-repository --repository-name $REPO_NAME --region $AWS_REGION
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('/var/lib/jenkins/workspace/10_mic_frontend/') {  // Change 'src' if your Dockerfile is elsewhere
                    sh '''
                    echo "Building Docker image..."
                    docker build -t $REPO_NAME:$IMAGE_TAG .
                    '''
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh '''
                echo "Tagging and pushing Docker image..."
                docker tag $REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$IMAGE_TAG
                docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$IMAGE_TAG
                '''
            }
        }
    }
}
