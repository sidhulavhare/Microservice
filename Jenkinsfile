pipeline {
    agent any

    stages {
        stage('EKS Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    caCertificate: '',
                    clusterName: 'EKS-1',
                    contextName: '',
                    credentialsId: 'k8s-token',
                    namespace: 'webapps',
                    serverUrl: 'https://4F5AECD6E44A2ADCC46E579237B5B6AC.gr7.ap-south-1.eks.amazonaws.com'
                ]]) {
                    sh '''
                        kubectl apply -f k8s/adservice.yml -n webapps

                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    caCertificate: '',
                    clusterName: 'EKS-1',
                    contextName: '',
                    credentialsId: 'k8s-token',
                    namespace: 'webapps',
                    serverUrl: 'https://4F5AECD6E44A2ADCC46E579237B5B6AC.gr7.ap-south-1.eks.amazonaws.com'
                ]]) {
                    sh '''
                        kubectl get all -n webapps
                    '''
                }
            }
        }
    }
}
