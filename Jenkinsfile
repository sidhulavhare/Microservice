pipeline {
    agent any

    environment {
        AWS_REGION     = "ap-south-1"
        AWS_ACCOUNT_ID = "427601800855"
        REPO_NAME      = "checkoutservice"
        IMAGE_TAG      = "" // Will be set dynamically from last commit
    }

    stages {
        stage('Clean Workspace'){
            steps {
                echo 'Cleaning the workspace before starting the build...'
                deleteDir()
            }
        }

        stage('Checkout Source Code') {
            steps {
                echo 'Checking out source code...'
                git branch: 'checkoutservice', url: 'https://github.com/sidhulavhare/Microservice.git'
            }
        }

        stage('Set Build Tag') {
            steps {
                script {
                    // Get short commit SHA as IMAGE_TAG
                    env.IMAGE_TAG = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    echo "Using IMAGE_TAG=${env.IMAGE_TAG}"
                }
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
                // Use current workspace instead of hardcoded path
                script {
                    echo "Building Docker image with tag $IMAGE_TAG..."
                    sh "docker build -t $REPO_NAME:$IMAGE_TAG ${env.WORKSPACE}"
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
