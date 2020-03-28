pipeline {
   agent any
   environment {
      LAST_BUILD_FILE = 'last_built_commit'
      CURRENT_COMMIT = sh (
         script: "git rev-parse HEAD",
         returnStdout: true
      )
      COMMIT_DELTA = sh (
         script: "git rev-list --count ${LAST_BUILD}..HEAD",
         returnStdout: true
      )
   }
   stages {
      stage('Decide Whether to Build') {
         steps {
            script {
               if (!fileExists ("${LAST_BUILD_FILE}")) {
                  writeFile(
                     file: "${LAST_BUILD_FILE}",
                     text: "${CURRENT_COMMIT}"
                  )
               }
            }
            LAST_BUILD = readFile "${LAST_BUILD_FILE}"
            COMMIT_DELTA = sh (
               script: "git rev-list --count ${LAST_BUILD}..HEAD",
               returnStdout: true
            )
         }
      }
      stage('Build') {
         steps {
            echo 'Building...'
            echo "${COMMIT_DELTA}"
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
