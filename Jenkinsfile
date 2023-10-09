pipeline {
  agent any

  parameters {
      choice(name: 'ENVIRONMENT', choices: ['dev', 'test', 'prod'], description: 'Choose environment')
  }
 
  environment {
      IMAGE_TAG = "xavnono/python_app"
      DOCKER_HUB_USER = "xavnono"
      DOCKER_HUB_PAT = "Xavi0501!"
  }

  stages {

//>>>>> Récupérer l’image sur le docker hub <<<<<//
      stage('Hub_Docker_Pull') {
          steps {
              sh 'docker pull xavnono/python_app:latest'
          }
      }

//>>>>> DEV "Dans une VM sur le cloud => simple docker run" <<<<<//
      stage('RUN_DEV') {
        when {
            expression {params.ENVIRONMENT == 'dev'}
        }
          steps {
              sh 'docker container run -d -p 8888:8000 --name python_app_dev xavnono/python_app:latest'
          }
      }

//>>>>> TEST "Dans une VM => Docker compose pour test" <<<<<//        
      stage('Scout_TEST') {
        when {
            expression {params.ENVIRONMENT == 'test'}
    //     }
    //       steps {
    //           sh 'docker-compose up -d'
    //           sleep 30
    //           sh 'docker-compose exec scout scout python_app:8888'
    //           sh 'docker-compose down'
    //       }
    //   }

             steps {
                // Install Docker Scout
                sh 'curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s -- -b /usr/local/bin'
                
                // Log into Docker Hub
                sh 'echo $DOCKER_HUB_PAT | docker login -u $DOCKER_HUB_USER --password-stdin'

                // Analyze and fail on critical or high vulnerabilities
                sh 'docker-scout cves $IMAGE_TAG --exit-code --only-severity critical,high'
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