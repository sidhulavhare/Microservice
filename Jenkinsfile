pipeline {
    agent any

    environment {
        AWS_REGION     = "ap-south-1"
        AWS_ACCOUNT_ID = "427601800855"
        REPO_NAME      = "checkoutservice"
        IMAGE_TAG      = "temp"   // placeholder (must NOT be empty)
    }

    stages {

        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Checkout Source Code') {
            steps {
                git branch: 'checkoutservice',
                    url: 'https://github.com/sidhulavhare/Microservice.git'
            }
        }

        stage('Set Image Tag (Last Commit Message)') {
            steps {
                script {
                    def commitMsg = sh(
                        script: "git log -1 --pretty=%s",
                        returnStdout: true
                    ).trim()

                    def shortSha = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()

                    // sanitize commit message
                    commitMsg = commitMsg.replaceAll("[^a-zA-Z0-9_.-]", "-")

                    // fallback if commit message becomes empty
                    env.IMAGE_TAG = commitMsg ? commitMsg : shortSha

                    echo "Docker image tag: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('AWS Login & ECR') {
            steps {
                withAWS(credentials: 'aws-jenkins-credentials', region: AWS_REGION) {
                    sh """
                    aws ecr get-login-password --region $AWS_REGION |
                    docker login --username AWS --password-stdin \
                    $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

                    aws ecr describe-repositories --repository-names $REPO_NAME \
                    || aws ecr create-repository --repository-name $REPO_NAME
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $REPO_NAME:$IMAGE_TAG .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                FULL_TAG=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$IMAGE_TAG
                docker tag $REPO_NAME:$IMAGE_TAG $FULL_TAG
                docker push $FULL_TAG
                """
            }
        }
    }

    post {
        success {
            echo "✅ Image pushed: ${IMAGE_TAG}"
        }
        always {
            deleteDir()
        }
    }
}
