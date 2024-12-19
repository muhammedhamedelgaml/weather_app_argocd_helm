pipeline {
    agent any

    environment{
    ARGOCD_SERVER = "localhost:8082" 
    APP_NAME = "weather"
    IMAGE_NAME = "muhammedhamedelgaml/app_python"
    IMAGE_TAG = "31"

    }
    

    stages {        
        // stage('Build image') {
        //     steps {
        //         script {
        //               echo "BUILD DONE"
        //               sh ' docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} . '
        //         }
        //     }
        // }

        // stage('Push image') {
        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {   
        //             sh '''
        //                 docker login --username $USERNAME --password $PASSWORD
        //                 docker push ${IMAGE_NAME}:${BUILD_NUMBER}

        //             '''

        //         }
        //     }
        // }

        stage('Update Image Tag') {
            steps {
                script {
                    sh """
                        sed -i 's|image:.*|image: $IMAGE_NAME:$IMAGE_TAG|' argocd/deployment.yml
                    """
                    // Commit the changes to Git
                    sh """
                        git checkout main
                        git add argocd/deployment.yml
                        git commit -m "Update image tag to $IMAGE_TAG"
                        git push origin main
                    """
                }
            }
        }        

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
