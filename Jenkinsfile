pipeline {
    agent any

    // Global variables
    environment {
        AWS_REGION     = "ap-south-1"
        AWS_ACCOUNT_ID = "427601800855"
        REPO_NAME      = "paymentservice"
    }

    stages {

        // 1️⃣ Clean old files
        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        // 2️⃣ Clone Git repository
        stage('Checkout Code') {
            steps {
                git branch: 'paymentservice',
                    url: 'https://github.com/sidhulavhare/Microservice.git'
            }
        }

        // 3️⃣ Get last commit message and use it as Docker tag
        stage('Set Image Tag') {
            steps {
                script {
                    // Get last commit message
                    def commitMsg = sh(
                        script: "git log -1 --pretty=%s",
                        returnStdout: true
                    ).trim()

                    // Convert to Docker-safe tag
                    env.IMAGE_TAG = commitMsg.replaceAll('[^a-zA-Z0-9_.-]', '-')

                    echo "Image Tag: ${env.IMAGE_TAG}"
                }
            }
        }

        // 4️⃣ Login to AWS ECR
        stage('AWS Login') {
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

        // 5️⃣ Build Docker image
        stage('Build Image') {
            steps {
                sh """
                docker build -t ${REPO_NAME}:${IMAGE_TAG} .
                """
            }
        }

        // 6️⃣ Push Docker image to ECR
        stage('Push Image') {
            steps {
                sh """
                IMAGE_URI=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}:${IMAGE_TAG}
                docker tag ${REPO_NAME}:${IMAGE_TAG} \$IMAGE_URI
                docker push \$IMAGE_URI
                """
            }
        }
    }

    // Always clean workspace after build
    post {
        always {
            deleteDir()
        }
    }
}
