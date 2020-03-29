buildingPipeline = {
   try {
      stage('Build') {
         sh 'mvn clean'
         sh 'mvn compile'
      }
   } catch (err) {
      slackSend (color: '#FF0000', message: "BUILD FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). Build errors are caused by malformed code, run the project locally to see build errors."
      )
   }

   try {
      stage('Test') {
         sh('mvn test')
      }
   } catch (err) {
      slackSend (color: '#FF0000', message: "TESTS FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). See build log for failed test or run 'mvn test' locally.")
      jobFailure = true
   }

   try {
      stage('Package') {
         echo 'Packaging...'
         sh 'mvn package'
      }
   } catch (err) {
      slackSend (color: '#FF0000', message: "PACKAGE FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). See build log for failure details.")
   }

   if (env.BRANCH_NAME == "master") {
      try {
         stage('Deploy') {
            echo 'Deploying...'
            sh 'mvn deploy'
         }
      } catch (err) {
         slackSend (color: '#FF0000', message: "DEPLOY FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). See build log for failure details.")
      }
   }
}

nonBuildingPipeline = {
   stage('No Actions') {
      echo("Current pipeline configured to build once every 8 commits")
   }
}

node {
   checkout scm
   String lastBuildFile = '/lastBuiltCommit.txt'
   String currentCommit = gitRevParse("HEAD")
   String lastBuild = getLastBuildHash(lastBuildFile)
   int commitDelta = getCommitDelta(lastBuild, currentCommit)

   stage("Debug") {
      echo "${commitDelta}"
   }

   if (commitDelta >= 8) {
      buildingPipeline()
      writeFileContents(lastBuildFile, currentCommit)
   } else {
      nonBuildingPipeline()
   }
}

String getLastBuildHash(String lastBuildPath) {
   String lastHash = getFileContents(lastBuildPath)
   if (lastHash == "") {
      writeFileContents(lastBuildPath, gitRevParse("HEAD"))
      lastHash = getFileContents(lastBuildPath)
   }
   return lastHash
}

int getCommitDelta(String earlier, String later) {
   return sh (
      script: "git rev-list --count ${earlier}..${later}",
      returnStdout: true
   )
}

String gitRevParse(String object) {
   return sh (
      script: "git rev-parse ${object}",
      returnStdout: true
   ).trim()
}

String getFileContents(String path) {
   try {
      return sh (
         script: "cat ${path}",
         returnStdout: true
      ).trim()
   } catch (err) {
      return ""
   }
}

void writeFileContents(String path, String contents) {
   sh("echo ${contents} >> ${path}")
}