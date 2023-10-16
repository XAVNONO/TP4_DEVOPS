pipeline {
  agent any

  parameters {
      choice(name: 'ENVIRONMENT', choices: ['dev', 'test', 'prod'], description: 'Choose environment')
  }
 
  environment {
      IMAGE_TAG = "${env.PYTHONAPP}"
      DOCKER_HUB_USER = "${env.MONID}"
      DOCKER_HUB_PAT = "${env.TOKHUBDOCKER}"
      DOCKER_HUB_MAIL = "${env.MONMAIL}"
  }

  stages {

//>>>>> DEV "Dans une VM sur le cloud => simple docker run" <<<<<//
      stage('RUN_DEV') {
        when {
            expression {params.ENVIRONMENT == 'dev'}
        }
          steps {
              // Récupération de l'image applicative provenant de Hub Docker 
              sh 'docker pull xavnono/python_app:latest'
              // Lancement du containeur applicatif
              sh 'docker container run -d -p 8888:8000 --name python_app_dev xavnono/python_app:latest'
              // Check état containeur "exited"
              sh 'while [ "$(docker inspect -f "{{.State.Status}}" python_app_dev)" != "exited" ]; do sleep 1; done'
              // Nettoyage containeur
              sh 'docker rm python_app_dev'
            }
          }

//>>>>> TEST "Dans une VM => Docker compose pour test" <<<<<//        
      stage('Test_Scout') {
        when {
            expression {params.ENVIRONMENT == 'test'}
        }
          steps {
              // Lancement containeur Docker Scout
              sh 'docker-compose up'
              // Check état containeur "exited"
              sh 'while [ "$(docker inspect -f "{{.State.Status}}" tp4-poei_scout-cli_1)" != "exited" ]; do sleep 1; done'
              // Nettoyage containeur
              sh 'docker-compose down'
            }
          }

//>>>>> PROD "Déploiement kubernetes" <<<<<//        
      stage('Deploy_PROD') {
        when {
            expression {params.ENVIRONMENT == 'prod'}
        }
          steps {
              // création de secret sur kubernetes
              // sh '''
              //     kubectl create secret docker-registry REGCRED --docker-server=https://hub.docker.com/repository/docker/xavnono /
              //       --docker-username=${DOCKER_HUB_USER} /
              //       --docker-password=${DOCKER_HUB_PAT} /
              //       --docker-email=${DOCKER_HUB_MAIL}
              //    '''
              sh '''
                    kubectl create secret docker-registry REGCRED --docker-username=${DOCKER_HUB_USER}
                    --docker-password=${DOCKER_HUB_PAT} --docker-email=${DOCKER_HUB_MAIL}
                 '''
            
              // Déploiement en réplica 3
              sh 'kubectl apply -f deployment.yaml'
              // Vérification du succés du déploiement
              sh 'kubectl rollout status deployment/python_app_deployment'
          }
        }
      }
    }
