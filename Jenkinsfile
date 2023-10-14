pipeline {
  agent any

  parameters {
      choice(name: 'ENVIRONMENT', choices: ['dev', 'test', 'prod'], description: 'Choose environment')
  }
 
  environment {
      IMAGE_TAG = "xavnono/python_app"
      DOCKER_HUB_USER = "xavnono"
      DOCKER_HUB_PAT = "dckr_pat_4J2oyGlyWzsg8HafBxz4YnTOqhQ"
  }

  stages {

//>>>>> Récupérer l’image sur le docker hub <<<<<//
    //   stage('Hub_Docker_Pull') {
    //       steps {
    //           sh 'docker pull xavnono/python_app:latest'
    //       }
    //   }

//>>>>> DEV "Dans une VM sur le cloud => simple docker run" <<<<<//
      stage('RUN_DEV') {
        when {
            expression {params.ENVIRONMENT == 'dev'}
        }
          steps {
              sh 'docker pull xavnono/python_app:latest'
              sh 'docker container run -d -p 8888:8000 --name python_app_dev xavnono/python_app:latest'
          }
      }

//>>>>> TEST "Dans une VM => Docker compose pour test" <<<<<//        
      stage('Test_Scout') {
        when {
            expression {params.ENVIRONMENT == 'test'}
        }
          steps {
              // lancement containeur Docker Scout
              sh 'docker-compose up -d --force-recreate'
              sh 'docker-compose down'
            }
        }

//>>>>> PROD "Déploiement kubernetes" <<<<<//        
      stage('Deploy_PROD') {
        when {
            expression {params.ENVIRONMENT == 'prod'}
        }
          steps {
              sh 'kubectl apply -f deployment.yaml'
              sh 'kubectl rollout status deployment/python_app_deployment'
          }
      }
  }
}