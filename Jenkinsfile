pipeline {

    environment {
        IMAGE_NAME = "unity3dgame"
        //IMAGE_TAG = "ajc-2.1"
        /*STAGING = "omar-ajc-staging-env"
        PRODUCTION = "omar-ajc-prod-env"*/
        //USERNAME = "omarpiotrdeveloper"
        CONTAINER_NAME = "unity3dgame"
        EC2_PRODUCTION_HOST = "omar-production.ddns.net"
    }

    agent none

    stages{

       stage ('Build Image')
       {
           agent any
           steps 
           {
               withCredentials([usernamePassword(credentialsId: 'dockerhub_password', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')])
               {
                   catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        script  {sh 'docker build -t $USERNAME/$IMAGE_NAME:$BUILD_TAG .' }
                   }
               }
           }    
       }

       stage ('Run test container') {
           agent any
           steps {
               withCredentials([usernamePassword(credentialsId: 'dockerhub_password', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')])
               {
                   catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        script{
                            sh '''
                                docker stop $CONTAINER_NAME || true
                                docker rm $CONTAINER_NAME || true
                                docker run --name $CONTAINER_NAME -d -p 80:80 $USERNAME/$IMAGE_NAME:$BUILD_TAG
                                sleep 5
                            '''
                        }
                   }
               }
           }
       }

       stage ('Test container') {
           agent any
           steps {
               script{
                   sh '''
                       curl http://localhost | grep -iq "ClickAndDestroy"
                   '''
               }
           }
       }

      stage ('save artifact') {
           agent any
           steps {
               withCredentials([usernamePassword(credentialsId: 'dockerhub_password', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')])
               {
                   catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        script{
                            sh '''
                                docker login -u $USERNAME -p $PASSWORD
                                docker push $USERNAME/$IMAGE_NAME:$BUILD_TAG
                                # docker stop $CONTAINER_NAME || true
                                # docker rm $CONTAINER_NAME || true
                                # docker rmi $USERNAME/$IMAGE_NAME:$BUILD_TAG
                            '''
                        }
                   }
               }
           }
       }

       /*

       stage('Deploy app on EC2-cloud Production ') {
            agent any
            when{
                expression{ GIT_BRANCH == 'origin/master'}
            }
            steps{
                withCredentials([sshUserPrivateKey(credentialsId: "ec2_prod_private_key", keyFileVariable: 'keyfile', usernameVariable: 'NUSER')]) {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        script{
                            timeout(time: 15, unit: "MINUTES") {
                                input message: 'Do you want to approve the deploy in production?', ok: 'Yes'
                            }						
                            sh'''
                                apk update
                                which ssh-agent || ( apk add openssh-client )
                                eval $(ssh-agent -s)
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker stop $CONTAINER_NAME || true
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker rm $CONTAINER_NAME || true
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker rmi $USERNAME/$IMAGE_NAME:$BUILD_TAG || true
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker run --name $CONTAINER_NAME -d -e PORT=5000 -p 5000:5000 $USERNAME/$IMAGE_NAME:$BUILD_TAG
                            '''
                        }
                    }
                }
            }
        }*/

        /******************************************************/

       /*stage('Push image in staging and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }  
            steps {
                script {
                   sh '''
                       heroku container:login
                       heroku create $STAGING || echo "project already exist"
                       heroku container:push -a $STAGING web
                       heroku container:release -a $STAGING web
                    '''
                }
            }
        }*/

        /*stage('Push image in Prod and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }  
            steps {
                script {
                   sh '''
                       heroku container:login
                       heroku create $PRODUCTION || echo "project already exist"
                       heroku container:push -a $PRODUCTION web
                       heroku container:release -a $PRODUCTION web
                    '''
                }
            }
        }  */      
    
        /*
        stage('Deploy app on EC2-cloud Production') {
            agent any
            when{
                expression{ GIT_BRANCH == 'origin/master'}
            }
            steps{
                withCredentials([sshUserPrivateKey(credentialsId: "ec2_prod_private_key", keyFileVariable: 'keyfile', usernameVariable: 'NUSER')]) {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        script{ 
                            timeout(time: 15, unit: "MINUTES") {
                            input message: 'Do you want to approve the deploy in production?', ok: 'Yes'
                            }
                            sh'''
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker stop $CONTAINER_NAME || true
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker rm $CONTAINER_NAME || true
                                ssh -o StrictHostKeyChecking=no -i ${keyfile} ${NUSER}@${EC2_PRODUCTION_HOST} docker run --name $CONTAINER_NAME -d -e PORT=5000 -p 5000:5000 $USERNAME/$IMAGE_NAME:$BUILD_TAG
                            '''
                        }
                    }
                }
            }
        }*/

         /******************************************************/

    }

   
    /*
    post {
        success{
            slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        failure {
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    }*/

}
