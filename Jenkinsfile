pipeline {
    agent any

    environment {
        AWS_REGION     = "ap-south-1"
        AWS_ACCOUNT_ID = "427601800855"
        REPO_NAME      = "checkoutservice"
        IMAGE_TAG      = "" // Will be set dynamically from last commit message
    }

    stages {
        stage('Clean Workspace') {
            steps {
                echo 'Cleaning the workspace before starting...'
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
                    // Get the last commit message and sanitize it for Docker tag
                    def commitMsg = sh(script: "git log -1 --pretty=%s", returnStdout: true).trim()
                    // Replace spaces and special characters with dashes
                    env.IMAGE_TAG = commitMsg.replaceAll("[^a-zA-Z0-9_.-]", "-")
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
                dir("${WORKSPACE}") {
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
                FULL_TAG=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$IMAGE_TAG
                docker tag $REPO_NAME:$IMAGE_TAG $FULL_TAG
                docker push $FULL_TAG
                '''
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace..."
            deleteDir()
        }
        success {
            echo "Docker image pushed successfully with tag: ${IMAGE_TAG}"
        }
        failure {
            echo "your Build failed!"
        }
    }
}
