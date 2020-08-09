pipeline {
   agent any

   stages {
      stage('Verify Branch') {
         steps {
            echo "$GIT_BRANCH"
         }
      }
      stage('Checkout') {
         steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/sasemb/azure-voting-app-redis.git']]])
                                 
            }
      }
      stage('Docker build stage') {
          steps {
              sh '''cd $WORKSPACE/azure-vote
                      docker images -a
                      docker build -t azure-vote-front . 
                      docker images -a 
                      echo "Docker build image succesfully" '''
          }
      }
      stage('Start test app') {
         steps {
            sh '''
               docker-compose up -d
               
            '''
         }
         post {
            success {
               echo "App started successfully :)"
            }
            failure {
               echo "App failed to start :("
            }
         }
      }
      stage('Run Tests') {
         steps {
            sh '''
               pytest ./tests/test_sample.py
            '''
         }
      }
      stage('Stop test app') {
         steps {
           sh '''
               docker-compose down
            ''' 
         }
      }
      stage('Checking container vulnerabilities'){
          steps{
              trivy image azure-vote-front
          }
      }
      stage('Deploy to QA') {
         environment {
            ENVIRONMENT = 'qa'
         }
         steps {
            echo "Deploying to ${ENVIRONMENT}"
            acsDeploy(
               azureCredentialsId: "jenkins_demo",
               configFilePaths: "**/*.yaml",
               containerService: "${ENVIRONMENT}-demo-cluster | AKS",
               resourceGroupName: "${ENVIRONMENT}-demo",
               sshCredentialsId: ""
            )
         }
      }
      stage('Approve PROD Deploy') {
         when {
            branch 'master'
         }
         options {
            timeout(time: 1, unit: 'HOURS') 
         }
         steps {
            input message: "Deploy?"
         }
         post {
            success {
               echo "Production Deploy Approved"
            }
            aborted {
               echo "Production Deploy Denied"
            }
         }
      }
      stage('Deploy to PROD') {
         when {
            branch 'master'
         }
         environment {
            ENVIRONMENT = 'prod'
         }
         steps {
            echo "Deploying to ${ENVIRONMENT}"
            acsDeploy(
               azureCredentialsId: "jenkins_demo",
               configFilePaths: "**/*.yaml",
               containerService: "${ENVIRONMENT}-demo-cluster | AKS",
               resourceGroupName: "${ENVIRONMENT}-demo",
               sshCredentialsId: ""
            )
         }
      }
   }
}