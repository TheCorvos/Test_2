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
        stage('Test') { 
            steps {
                sh './jenkins/scripts/test.sh' 
            }
        }
        stage('Deliver') { 
            steps {
                sh './jenkins/scripts/deliver.sh' 
                input message: 'Finished using the web site? (Click "Proceed" to continue)' 
                sh './jenkins/scripts/kill.sh' 
            }
        }
		stage ('SonarQube Scan'){
            steps {
                withSonarQubeEnv('SonarQube') {
                    withMaven (
                        maven: 'maven 3.3.9',
                        mavenSettingsConfig: 'cc86690e-095d-4714-92b2-b61861241c7a'){
                        sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar '
                    }
                } // SonarQube taskId is automatically attached to the pipeline context
            }
        }
        stage ('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
                    script {
                        def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
    }
}
