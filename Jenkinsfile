pipeline {
    agent {
        docker {
            image 'node:6-alpine'
            args '-p 3000:3000'
        }
    }
    environment {
        CI = 'true' 
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
      stage('SonarQube analysis') { 
      when {
                branch 'master'
            }
      steps {
        withSonarQubeEnv('SonarQube') { 
         sh 'mvn sonar:sonar'
         }
        }
    }
     stage("SonarQube Quality Gate") { 
      when {
                branch 'master'
            }
      steps {
        timeout(time: 10, unit: 'MINUTES') { 
        script {
            sleep 120
           def qg = waitForQualityGate() 
           if (qg.status != 'OK') {
             error "Pipeline aborted due to quality gate failure: ${qg.status}"
           }
           }
        }
        }
    }
		
    }
}
