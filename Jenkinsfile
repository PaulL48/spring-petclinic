node {
   checkout scm
   String lastBuildFile = '/lastBuiltCommit.txt'
   String currentCommit = gitRevParse("HEAD")
   String lastBuild = getLastBuildHash(lastBuildFile)
   int commitDelta = getCommitDelta(lastBuild, currentCommit)

   if (commitDelta < 8) {

   } else {

   }
   buildingPipeline()
   // stage('Build') {
   //    echo 'Building...'
   //    echo "${currentCommit}, ${lastBuild}, ${commitDelta}"
   //    sh 'mvn clean'
   //    sh 'mvn compile'
   // }

   // stage('Test') {
   //    echo 'Testing...'
   //    sh 'mvn test'
   // }

   // stage('Package') {
   //    echo 'Packaging...'
   //    sh 'mvn package'
   // }

   // if (env.BRANCH_NAME == "master") {
   //    stage('Deploy') {
   //       echo 'Deploying...'
   //       sh 'mvn deploy'
   //    }
   // }
}

nonBuildingPipeline = {

}

buildingPipeline = {
   boolean jobFailure = false

   try {
      stage('Build') {
         echo 'Building...'
         echo "${currentCommit}, ${lastBuild}, ${commitDelta}"
         sh 'mvn clean'
         sh 'mvn compile'
      }
   } catch (err) {
      slackSend (
         color: '#FF0000', 
         message: "BUILD FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). Build errors are caused by malformed code, run the project locally to see build errors."
      )
      jobFailure = true
   }

   jobFailure |= ext_stage(
      name: 'Test',
      steps: {
         echo('Testing...')
         sh('mvn test')
      },
      postFailure: {
         slackSend (color: '#FF0000', message: "TESTS FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}). See build log for failed test or run 'mvn test' locally.")
      }
   )

   stage('Package') {
      echo 'Packaging...'
      sh 'mvn package'
   }

   if (env.BRANCH_NAME == "master") {
      stage('Deploy') {
         echo 'Deploying...'
         sh 'mvn deploy'
      }
   }
}

boolean ext_stage(String name, Closure steps, Closure postFailure) {
   try {
      stage(name) {
         steps()
      }
   } catch (err) {
      postFailure()
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