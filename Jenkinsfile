node {
   stage('Build') {
      echo 'Building...'
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

   //if (env.BRANCH_NAME == "master") {
      stage('Deploy') {
         when {
            branch 'master'
         }
         echo 'Deploying...'
         sh 'mvn deploy'
      }
   //}

}