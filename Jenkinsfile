pipeline {
    agent any
    environment {
        GIT_CREDENTIALS_ID = 'github-token'
        ARGOCD_SERVER = 'argocd-server-url'
        ARGOCD_USER = 'argocd-username'
        ARGOCD_PASSWORD = 'argocd-password'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/samskrutha/sketch-argocd.git', credentialsId: "${GIT_CREDENTIALS_ID}"
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
