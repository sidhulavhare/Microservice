pipeline {
    agent any

    environment {
        AWS_REGION     = "ap-south-1"
        AWS_ACCOUNT_ID = "427601800855"
        REPO_NAME      = "checkoutservice"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'checkoutservice',
                    url: 'https://github.com/sidhulavhare/Microservice.git'
            }
        }

        stage('Set Image Tag (Last Commit Message)') {
            steps {
                script {
                    def msg = sh(
                        script: "git log -1 --pretty=%s",
                        returnStdout: true
                    ).trim()

                    // make docker-safe tag
                    msg = msg.replaceAll('[^a-zA-Z0-9_.-]', '-')

                    env.IMAGE_TAG = msg

                    echo "Docker image tag: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('AWS Login & ECR') {
            steps {
                withAWS(credentials: 'aws-jenkins-credentials', region: AWS_REGION) {
                    sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin \
                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

                    aws ecr describe-repositories --repository-names ${REPO_NAME} || \
                    aws ecr create-repository --repository-name ${REPO_NAME}
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${REPO_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                IMAGE_URI=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}:${IMAGE_TAG}
                docker tag ${REPO_NAME}:${IMAGE_TAG} \$IMAGE_URI
                docker push \$IMAGE_URI
                """
            }
        }
    }

    post {
        always {
            deleteDir()
        }
    }
}
