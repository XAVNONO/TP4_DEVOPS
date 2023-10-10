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
      stage('Vulnérability_TEST') {
        when {
            expression {params.ENVIRONMENT == 'test'}
             }
           
          steps {
                // Build an image for scanning
                sh 'echo "FROM xavnono/python_app:latest" > Dockerfile'
                sh 'docker build --no-cache -t test/test-image:0.1 .'
                }
        }
      stage('Scan') {
          steps {
                // Scan the image
                prismaCloudScanImage ca: '',
                cert: '',
                dockerAddress: 'unix:///var/run/docker.sock',
                image: 'test/test-image*',
                key: '',
                logLevel: 'info',
                podmanPath: '',
                // The project field below is only applicable if you are using Prisma Cloud Compute Edition and have set up projects (multiple consoles) on Prisma Cloud.
                project: '',
                resultsFile: 'prisma-cloud-scan-results.json',
                ignoreImageBuildTime:true
                }
        }
    }
    post {
        always {
            // The post section lets you run the publish step regardless of the scan results
            prismaCloudPublish resultsFilePattern: 'prisma-cloud-scan-results.json'
               }
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