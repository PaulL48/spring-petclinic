pipeline {
   agent any
   environment {
      LAST_BUILD_FILE = 'last_built_commit'
      CURRENT_COMMIT = sh (
         script: "git rev-parse HEAD",
         returnStdout: true
      )
      LAST_BUILD = getLastBuild(LAST_BUILD_FILE, CURRENT_COMMIT)
      COMMIT_DELTA = getCommitDelta(LAST_BUILD, CURRENT_COMMIT)
   }
   stages {
      stage('Build') {
         steps {
            echo 'Building...'
            echo "${CURRENT_COMMIT}"
            echo "${LAST_BUILD}"
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

def getLastBuild(lastBuildPath, currentCommit) {
   if (!fileExists (lastBuildPath)) {
      writeFile (
         file: lastBuildPath,
         text: currentCommit
      )
   }
   return readFile (lastBuildPath).trim()
}

def getCommitDelta(earlier, later) {
   return sh (
      script: "git rev-list --count ${earlier}..${later}",
      returnStdout: true
   )
}
