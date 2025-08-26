pipeline {
    agent any

    environment {
        AWS_REGION     = "ap-south-1"                       // your AWS region
        AWS_ACCOUNT_ID = "427601800855"                    // your AWS account ID
        REPO_NAME      = "my-app-repo"                     // ECR repository name
        IMAGE_TAG      = "${BUILD_NUMBER}"                 // unique tag for each build
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/sidhulavhare/Microservice.git'
            }
        }

        stage('AWS Login & Create ECR Repository') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                                  credentialsId: 'aws-jenkins-credentials']]) {
                    script {
                        sh """
                        echo "Logging into AWS..."
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

                        echo "Checking if ECR repository exists..."
                        aws ecr describe-repositories --repository-names ${REPO_NAME} --region ${AWS_REGION} || \
                        aws ecr create-repository --repository-name ${REPO_NAME} --region ${AWS_REGION}
                        """
                    }
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

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    sh """
                    docker tag ${REPO_NAME}:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}:${IMAGE_TAG}
                    docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Docker image pushed to ECR successfully: ${REPO_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "Pipeline failed. Check logs for errors."
        }
    }
}
