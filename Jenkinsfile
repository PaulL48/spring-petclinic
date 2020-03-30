buildingPipeline = { boolean bisectAvailable, String currentCommit, String lastSuccessfulBuild , String lastBuildPath ->
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
      if (jobSuccess) {
         stage('Test') {
            sh('mvn test')
         }
      }
   } catch (err) {
      stage('Log Test Failure') {
         reportFailedCommit(
            findFailedCommit(bisectAvailable, lastSuccessfulBuild, currentCommit),
            "TESTS FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). " +
            "See build log for failed test or run 'mvn test' locally."
         )
      }
      error("Test stage failed")
      jobSuccess = false
   }

   try {
      if (jobSuccess) {
         stage('Package') {
            sh('mvn package')
         }
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
   String lastBuildFile = '/last--build.txt'
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

/**
 * Return the last successful build hash or empty string if there is none
 */
String getLastBuildHash(String lastBuildPath) {
   return getFileContents(lastBuildPath)
}

/**
 * Return the number of commits between two commits, including
 * the right endpoint
 */
int getCommitDelta(String earlier, String later) {
   String result = sh (
      script: "git rev-list --count ${earlier}..${later}",
      returnStdout: true
   ).trim()
   return Integer.parseInt(result)
}

/**
 * If bisection is available, perform bisect between current and last successful
 * commit. Otherwise return the current commit
 */
String findFailedCommit(boolean bisectAvailable, String lastSuccessfulBuild, String currentCommit) {
   if (bisectAvailable) {
      return gitBisect(lastSuccessfulBuild, currentCommit)
   } else {
      return currentCommit
   }
}

/**
 * Send Slack notification containing a message and the commit that is failing
 */
void reportFailedCommit(String badCommit, String message) {
   slackSend(color: '#FF0000', message: message)
   slackSend(color: '#FF0000', message: "Failing commit hash: ${badCommit}")
   slackSend(color: '#FF0000', message: "Access at: https://github.com/PaulL48/spring-petclinic/commit/${badCommit}")
}

/**
 * Performs a git bisect and return the resulting commit
 */
String gitBisect(String stable, String breaking) {
   sh("git bisect start ${breaking} ${stable}")
   sh("git bisect run mvn clean test")

   // Git bisect does not make it easy to extract output
   // We may be on the last good or the first bad commit so we need to test
   int testStatus = sh (
      script: "mvn clean test",
      returnStatus: true
   )

   // Advance head by 1 commit if tests pass
   if (testStatus == 0) {
      sh("git reset HEAD@{1}")
   }

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
