buildingPipeline = {
   boolean jobSuccess = true

   try {
      stage('Build') {
         sh('mvn clean')
         sh('mvn compile')
      }
   } catch (err) {
      slackSend(color: '#FF0000', message: "BUILD FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). Build errors are caused by malformed code, run the project locally to see build errors.")
      jobSuccess = false
   }

   try {
      stage('Test') {
         sh('mvn test')
      }
   } catch (err) {
      slackSend(color: '#FF0000', message: "TESTS FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). See build log for failed test or run 'mvn test' locally.")
      jobSuccess = false
   }

   try {
      stage('Package') {
         sh('mvn package')
      }
   } catch (err) {
      slackSend(color: '#FF0000', message: "PACKAGE FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). See build log for failure details.")
      jobSuccess = false
   }

   if (env.BRANCH_NAME == "master") {
      try {
         stage('Deploy') {
            sh('mvn deploy')
         }
      } catch (err) {
         slackSend (color: '#FF0000', message: "DEPLOY FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). See build log for failure details.")
         jobSuccess = false
      }
   }

   if (jobSuccess) {
      slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
   }
}

nonBuildingPipeline = {
   stage('No Actions') {
      echo("Current pipeline configured to build once every 8 commits")
   }
}

node {
   checkout scm
   String lastBuildFile = '/lastBuiltHash.txt'
   String currentCommit = getGitObjectHash("HEAD")
   String lastBuild = getLastBuildHash(lastBuildFile)
   int commitDelta = getCommitDelta(lastBuild, currentCommit)

   stage("Pipeline Status") {
      echo("Pipeline is configured to build the project every 8 commits")
      echo("Last commit that was built: ${lastBuild}")
      echo("Current commit is ${commitDelta} commits ahead")
   }

   // If the previous build file is not yet created the commit delta will be 0
   // This is the only case where commit delta is 0 and therefore a build should take place
   if (commitDelta >= 8 || commitDelta == 0) {
      buildingPipeline()
      writeFileContents(lastBuildFile, currentCommit)
   } else {
      nonBuildingPipeline()
   }
}

String getLastBuildHash(String lastBuildPath) {
   String lastHash = getFileContents(lastBuildPath)
   if (lastHash == "") {
      writeFileContents(lastBuildPath, getGitObjectHash("HEAD"))
      lastHash = getFileContents(lastBuildPath)
   }
   return lastHash
}

int getCommitDelta(String earlier, String later) {
   return sh (
      script: "git rev-list --count ${earlier}..${later}",
      returnStdout: true
   ).trim()
}

String getGitObjectHash(String object) {
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
   sh("echo ${contents} > ${path}")
}