buildingPipeline = { boolean bisectAvailable, String currentCommit, String lastSuccessfulBuild , String lastBuildPath->
   boolean jobSuccess = true

   try {
      stage('Build') {
         sh('mvn clean')
         sh('mvn compile')
      }
   } catch (err) {
      reportFailedCommit(
         currentCommit,
         "BUILD FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). " + 
         "Build errors are caused by malformed code, run the project locally to see build errors."
      )
      jobSuccess = false
   }

   try {
      stage('Test') {
         sh('mvn test')
      }
   } catch (err) {
      reportFailedCommit(
         findFailedCommit(bisectAvailable, lastSuccessfulBuild, currentCommit),
         "TESTS FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). " +
         "See build log for failed test or run 'mvn test' locally."
      )
      jobSuccess = false
   }

   try {
      stage('Package') {
         sh('mvn package')
      }
   } catch (err) {
      reportFailedCommit(
         currentCommit,
         "PACKAGE FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). " +
         "See build log for failure details."
      )
      jobSuccess = false
   }

   if (jobSuccess) {
      stage('Storing Build Info') {
         writeFileContents(lastBuildPath, currentCommit)
         slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
   }
}

nonBuildingPipeline = {
   stage('No Actions') {
      echo("Current pipeline configured to build once every 8 commits")
      slackSend (color: '#0000FF', message: "BUILD NOT TRIGGERED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). Build is triggered every 8 commits")
   }
}

node {
   checkout scm
   String lastBuildFile = '/lbuild.txt'
   String currentCommit = getCurrentCommit()
   String lastSuccessfulBuild = getLastBuildHash(lastBuildFile)
   int deltaCommit = 0
   
   if (lastSuccessfulBuild != "") {
      deltaCommit = getCommitDelta(lastSuccessfulBuild, currentCommit)
   }

   stage("Pipeline Status") {
      echo("Pipeline is configured to build the project every 8 commits on master")
      echo("Last commit that was built: ${lastSuccessfulBuild}")
      echo("Current commit is ${deltaCommit} commits ahead")
   }

   if (env.BRANCH_NAME == "master") {
      if (deltaCommit >= 8 || deltaCommit == 0) {
         // If commit delta is greater than zero, bisection is available
         buildingPipeline((deltaCommit > 0), currentCommit, lastSuccessfulBuild, lastBuildFile)
      } else {
         nonBuildingPipeline()
      }
   }
}

String getLastBuildHash(String lastBuildPath) {
   return getFileContents(lastBuildPath)
}

int getCommitDelta(String earlier, String later) {
   String result = sh (
      script: "git rev-list --count ${earlier}..${later}",
      returnStdout: true
   ).trim()
   return Integer.parseInt(result)
}

String findFailedCommit(boolean bisectAvailable, String lastSuccessfulBuild, String currentCommit) {
   if (bisectionAvailable) {
      return gitBisect(lastSuccessfulBuild, currentCommit)
   } else {
      return currentCommit
   }
}

void reportFailedCommit(String badCommit, String message) {
   slackSend(color: '#FF0000', message: message)
   slackSend(color: '#FF0000', message: "Failing commit hash: ${badCommit}")
}

String gitBisect(String leftEndpoint, String rightEndpoint) {
   sh("git bisect start ${leftEndpoint} ${rightEndpoint}")
   sh("git bisect run mvn clean test")
   String badCommit = getCurrentCommit()
   sh("git bisect reset")
   return badCommit
}

String getCurrentCommit() {
   return sh (
      script: 'git log -1 --format="%H"',
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