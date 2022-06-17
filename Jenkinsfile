pipeline {
     environment {
       ID_DOCKER = "b3col3"
       IMAGE_NAME = "final-exam-1"
       IMAGE_TAG = "latest"
       STAGING = "${ID_DOCKER}-${IMAGE_NAME}-staging"
       PRODUCTION = "${ID_DOCKER}-${IMAGE_NAME}-production"
       DOCKERHUB_CREDENTIALS=credentials('dockerhub')
     }
     agent none
     stages {
         stage('Build image') {
             agent any
             steps {
                script {
                  sh 'docker build -t ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
             }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                  docker run -dit --name ${IMAGE_NAME} -p 80:80 ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG} 
                 '''
               }
            }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh '''
                 curl http://172.17.0.1 | grep -i "DIMENSION"
                '''
              }
           }
      }
      stage('Clean Container') {
          agent any
          steps {
             script {
               sh '''
                docker stop ${IMAGE_NAME}
                docker rm  ${IMAGE_NAME}
               '''
             }
          }
     }

     stage ('Login and Push Image on docker hub') {
          agent any
          steps {
             script {
               sh '''
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                docker push ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}
               '''
             }
          }
      }    
     
     stage('Push image in staging and deploy it') {
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
     }



     stage('Push image in production and deploy it') {
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
     }
  }
}