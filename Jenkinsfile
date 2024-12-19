pipeline {
    agent any

    environment{
    ARGOCD_SERVER = "localhost:8082" 
    APP_NAME = "weather"
    }
    

    stages {        
        stage('authenticate with ArgoCD') {
            steps {
                script {
               withCredentials([usernamePassword(credentialsId: 'argocdcred', passwordVariable: 'password', usernameVariable: 'username')]) {
                     sh '''
                     argocd login $ARGOCD_SERVER --username $username --password $password --insecure
                     '''
                }
            }
        }
}
        stage('create APP') {
            steps {
                script {
                    // Create the ArgoCD app
                    sh '''
                    argocd app create $APP_NAME \
                        --repo https://github.com/muhammedhamedelgaml/weather_app_argocd_helm.git\
                        --path argocd \
                        --dest-server https://kubernetes.default.svc \
                        --dest-namespace default
                    '''
                }
            }
        }

        stage('resync') {
            steps {
                script {
                    // Resync the ArgoCD app
                    sh 'argocd app sync weather3'
                }
            }
        }

    }

}
