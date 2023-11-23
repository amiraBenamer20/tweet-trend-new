pipeline {
    agent {
        node {
            label "build-slave"
        }
    }

environment {
    PATH = "/opt/apache-maven-3.9.5/bin:$PATH"
}
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean deploy'
            }
        }
        
         stage('SonarQube analysis') {
         environment {
          scannerHome = tool 'code-sonar-scanner'
        }
        
        steps{
        withSonarQubeEnv('sonarqube-server') { // If you have configured more than one global server connection, you can specify its name
          sh "${scannerHome}/bin/sonar-scanner"
        }
        }
      }
    }
}
