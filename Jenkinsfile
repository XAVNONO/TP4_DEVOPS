pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
   
    stages {

//>>>>> Récupérer l’image sur le docker hub <<<<<//
        stage('Push') {
            steps {
                sh 'docker pull xavnono/mypythonapp:latest'
            }
        }

//>>>>> DEV "Dans une VM sur le cloud => simple docker run" <<<<<//
        stage('Build_DEV') {
            steps {
                sh 'docker run -d --name mypythonapp-dev -p 8080:8088 xavnono/mypythonapp:latest'
            }
        }

//>>>>> TEST "Dans une VM => Docker compose pour test" <<<<<//        
        stage('Scout_TEST') {
            steps {
                sh 'docker-compose up -d'
                sleep 30
                sh 'docker-compose exec scout scout mypythonapp:80'
                sh 'docker-compose down'
            }
        }
        
//>>>>> PROD "Déploiement kubernetes" <<<<<//        
        stage('Deploy_PROD') {
            steps {
                sh 'kubectl apply -f deployment.yaml'
                sh 'kubectl rollout status deployment/mypythonapp-deployment'
            }
        }

    }
}