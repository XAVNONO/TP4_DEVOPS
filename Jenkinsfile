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
      DOCKER_REGISTRY = "${env.MONREGISTRE}"
  }

  stages {

//>>>>> DEV "Dans une VM sur le cloud => simple docker run" <<<<<//
      stage('RUN_DEV') {
        when {
          expression {params.ENVIRONMENT == 'dev'}
        }
          steps {
              // Récupération de l'image applicative provenant de Hub Docker 
              sh 'docker pull ${IMAGE_TAG}'
              // Lancement du containeur applicatif
              sh 'docker container run -d -p 9999:8000 --name python-app-dev ${IMAGE_TAG}'
              // Check état containeur "exited"
              sh 'while [ "$(docker inspect -f "{{.State.Status}}" python-app-dev)" != "exited" ]; do sleep 1; done'
              // Nettoyage containeur
              sh 'docker rm python-app-dev'
          }
      }

//>>>>> TEST "Dans une VM => Docker compose pour test" <<<<<//     
      stage('Clone_REPO_TEST') {
        when {
            expression {params.ENVIRONMENT == 'test'}
        }
      
          steps {              
              // Récupération du repository dans le container Jenkins
              git credentialsId: 'token-github' , url: 'https://github.com/XAVNONO/TP4_DEVOPS.git'
          }
      }

      stage('SCOUT_TEST') {
        when {
            expression {params.ENVIRONMENT == 'test'}
        }

          steps {
              // Positionnement dans le container jenkins
              sh 'cd /var/jenkins_home/workspace/TP4-DEVOPS'
              // Lancement containeur Docker Scout
              sh 'docker-compose up'
              // Check état containeur "exited"
              sh 'while [ "$(docker inspect -f "{{.State.Status}}" scout-test)" != "exited" ]; do sleep 1; done'
              // Nettoyage containeur
              sh 'docker-compose down'
          }
      }

//>>>>> PROD "Déploiement kubernetes" <<<<<//        
      stage('Clone_REPO_DEPLOY') {
        when {
            expression {params.ENVIRONMENT == 'prod'}
        }
      
          steps {              
              // Récupération du repository dans le container Jenkins
              git credentialsId: 'token-github' , url: 'https://github.com/XAVNONO/TP4_DEVOPS.git'
            }
          }
      
      stage('MINIKUBE_start') {
        when {
            expression {params.ENVIRONMENT == 'prod'}
        }
          steps {
              sh 'minikube start --driver=docker --force'
          }
      }

      stage('DEPLOY_PROD') {
        when {
            expression {params.ENVIRONMENT == 'prod'}
        }
          steps {
              // création de secret sur kubernetes
              sh '''
                  kubectl create secret docker-registry regcred \
                    --docker-server=${DOCKER_REGISTRY} \
                    --docker-username=${DOCKER_HUB_USER} \
                    --docker-password=${DOCKER_HUB_PAT} \
                    --docker-email=${DOCKER_HUB_MAIL}
                 '''
              script {
              // Déploiement en réplica 3
              sh 'kubectl apply -f deployment.yaml'
              // // Vérification du succés du déploiement
              sh 'kubectl rollout status deployment/python-app-deployment'
              }
          }
      }
  }
}
