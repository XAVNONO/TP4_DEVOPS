pipeline {
  agent any
  options {
      skipStagesAfterUnstable()
  }
 
  stages {

//>>>>> Récupérer l’image sur le docker hub <<<<<//
      stage('Pull') {
          steps {
              sh 'docker pull xavnono/python_app:latest'
          }
      }

//>>>>> DEV "Dans une VM sur le cloud => simple docker run" <<<<<//
      stage('RUN_DEV') {
          steps {
              sh 'docker container run -d -p 8888:8000 --name python_app_dev xavnono/python_app:latest'
          }
      }

//>>>>> TEST "Dans une VM => Docker compose pour test" <<<<<//        
      stage('Scout_TEST') {
          steps {
              sh 'docker-compose up -d'
              sleep 30
              sh 'docker-compose exec scout scout python_app:8088'
              sh 'docker-compose down'
          }
      }
      
//>>>>> PROD "Déploiement kubernetes" <<<<<//        
      stage('Deploy_PROD') {
          steps {
              sh 'kubectl apply -f deployment.yaml'
              sh 'kubectl rollout status deployment/python_app_deployment'
          }
      }

  }
}