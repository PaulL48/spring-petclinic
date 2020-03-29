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
   File commitFile = new File(lastBuildPath)
   if (!commitFile.exists()) {
      commitFile << getRevParse("HEAD")
   }

   return commitFile.text.trim()
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
   )
}