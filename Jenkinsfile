pipeline {
    agent any
    stages {
        stage('Git Check Out') {
            steps {
                script {
                    git branch: 'stage', url: 'https://github.com/sidhulavhare/Microservice.git'
                }
            }
        }

        stage('Adservice') {
            steps {
                script {
                    def awsAccountId = '851725270304' // Replace with your AWS Account ID
                    def region = 'us-east-1' // e.g., us-east-1, ap-south-1
                    def ecrRepo = "${awsAccountId}.dkr.ecr.${region}.amazonaws.com/adservice"
                    
                    withAWS(credentials: 'aws-ecr-cred', region: region) {
                        // Login to AWS ECR
                        bat "aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${ecrRepo}"
                        
                        // Building and pushing Docker image
                        dir('C:/var/lib/jenkins/workspace/10-Tier/src/adservice') {
                            bat "docker build -t ${ecrRepo}:latest ."
                            bat "docker push ${ecrRepo}:latest"
                            bat "docker rmi ${ecrRepo}:latest"
                        }
                    }
                }
            }
        }
    }
}
