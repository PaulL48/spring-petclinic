pipeline {
   agent any
   environment {
      LAST_BUILD = '62d58432cad547b10651d9e5e957d03935ab3696'
      COMMIT_COUNT = sh (
         // This gives the distance between two commits 1 higher than
         script: "git rev-list --count ${LAST_BUILD}..HEAD",
         returnStdout: true
      )
      COMMIT_DELTA = COMMIT_COUNT - 1
   }
   stages {
      stage('Build') {
         steps {
            echo 'Building...'
            echo "${COMMIT_COUNT}"
            sh 'mvn clean'
            sh 'mvn compile'

         }
         post {
            failure {
               slackSend (color: '#FF0000', message: "BUILD FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). Build errors are caused by malformed code, run the project locally to see build errors.")
            }
         }
      }
      stage('Test') {
         steps {
            echo 'Testing...'
            sh 'mvn test'
         }
         post {
            failure {
               slackSend (color: '#FF0000', message: "TESTS FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). See build log for failed test or run 'mvn test' locally.")
            }
         }
      }
      stage('Package') {
         
         steps {
            echo 'Packaging...'
            sh 'mvn package'
         }
         post {
            failure {
               slackSend (color: '#FF0000', message: "PACKAGE FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). See build log for failure details.")
            }
         }
      }
      stage('Deploy') {
         when {
            branch 'master'
         }
         steps {
            echo 'Deploying...'
            sh 'mvn deploy'
         }
         post {
            failure {
               slackSend (color: '#FF0000', message: "DEPLOY FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). See build log for failure details.")
            }
         }
      }
   }

   post {
      success {
         slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
      failure {
         slackSend (color: '#FF0000', message: "JOB FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}).")

      }
   }
}