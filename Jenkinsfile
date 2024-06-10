pipeline {
    agent any
    environment {
        GIT_CREDENTIALS_ID = 'github-token'
        ARGOCD_SERVER = 'argocd-server-url'
        ARGOCD_USER = 'argocd-username'
        ARGOCD_PASSWORD = 'argocd-password'
        AWS_REGION = 'eu-west-1'
        CLUSTER_NAME = 'docusketch-cluster'
        AWS_CREDENTIALS_ID = 'aws-credentials'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/samskrutha/sketch-argocd.git', credentialsId: "${GIT_CREDENTIALS_ID}"
            }
        }
        stage('Configure kubectl') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                    sh 'aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}'
                }
            }
        }
        stage('Apply Manifests') {
            steps {
                sh 'kubectl apply -f apps/sketch-web-app/deployment.yaml'
                sh 'kubectl apply -f apps/sketch-web-app/service.yaml'
            }
        }
        stage('Sync ArgoCD') {
            steps {
                sh """
                argocd login ${ARGOCD_SERVER} --username ${ARGOCD_USER} --password ${ARGOCD_PASSWORD} --insecure
                argocd app sync sketch-web-app
                """
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
