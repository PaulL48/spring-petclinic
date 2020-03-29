node {
   String lastBuildFile = 'lastBuiltCommit.txt'
   String currentCommit = gitRevParse("HEAD")
   String lastBuild = getLastBuildHash(lastBuildFile)
   int commitDelta = getCommitDelta(lastBuild, currentCommit)

   stage('Build') {
      echo 'Building...'
      echo "${currentCommit}, ${lastBuild}, ${commitDelta}"
      sh 'mvn clean'
      sh 'mvn compile'
   }

   stage('Test') {
      echo 'Testing...'
      sh 'mvn test'
   }

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


String getLastBuildHash(String lastBuildPath) {
   String lastHash = getFileContents(lastBuildPath)
   if (lastHash == "") {
      writeFileContents(lastBuildPath, gitRevParse("HEAD"))
   }

   return getFileContents(lastBuildPath).trim()
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
      def output = sh (
         script: "cat ${path}",
         returnStdout: true
      )
      return output
   } catch (err) {
      return ""
   }
}

void writeFileContents(String path, String contents) {
   sh("echo ${contents} >> ${path}")
}