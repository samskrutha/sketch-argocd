pipeline {
    agent any
    environment {
        GIT_CREDENTIALS_ID = 'github-token'
        ARGOCD_SERVER = 'a13972dab7cea41a69fcc7e2d763cf6b-1608553155.eu-west-1.elb.amazonaws.com'  
        AWS_REGION = 'eu-west-1'
        CLUSTER_NAME = 'docusketch-cluster'
        AWS_CREDENTIALS_ID = 'aws-credentials'
        ARGOCD_CREDENTIALS_ID = 'argocd-credentials' 
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
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                    sh 'kubectl apply -f apps/sketch-web-app/deployment.yaml'
                    sh 'kubectl apply -f apps/sketch-web-app/service.yaml'
                }
            }
        }
        stage('Sync ArgoCD') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${ARGOCD_CREDENTIALS_ID}", passwordVariable: 'ARGOCD_PASSWORD', usernameVariable: 'ARGOCD_USER')]) {
                    sh """
                    argocd login ${ARGOCD_SERVER} --username ${ARGOCD_USER} --password ${ARGOCD_PASSWORD} --insecure
                    argocd app sync sketch-web-app
                    """
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
