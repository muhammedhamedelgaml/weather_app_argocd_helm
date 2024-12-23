def COLOR_MAP = [
  'SUCCESS' : 'good',
  'FAILURE' : 'danger'
]

pipeline {
    agent any

    environment{
    ARGOCD_SERVER = "localhost:8082" 
    APP_NAME = "weather"
    IMAGE_NAME = "muhammedhamedelgaml/app_python"
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

        //stage('Update Image Tag using sed ') {
            // 1- using sed 
            // steps {
            //     script {
            //         sh """
            //             sed -i 's|image:.*|image: $IMAGE_NAME:$IMAGE_TAG|' argocd/deployment.yml
            //         """
            //         // Commit the changes to Git
            //         sh """
            //             git add argocd/deployment.yml
            //             git commit -m "Update image tag to $IMAGE_TAG"
            //             git push origin main
            //         """
            //     }
            // }
             
        //}        

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
                        --path helm/weathercharts \
                        --helm-set image=muhammedhamedelgaml/app_python:31 \
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
                    sh 'argocd app sync $APP_NAME '
                }
            }
        }

    }

    post {
        always {
            echo 'Pipeline finished! \n logging out from docker'
            sh 'docker logout'

              
            echo 'slack notification.'
            slackSend channel: '#cicdjenkins',
            color:  COLOR_MAP[currentBuild.currentResult],
            message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n for more info visit : ${env.BUILD_URL} " 
        
        }
    }

}
