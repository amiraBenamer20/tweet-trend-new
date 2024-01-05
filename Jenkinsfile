pipeline{
    agent   {
        node    {
            label "maven"
        }
    }

    
            
    environment  {
        PATH = "/opt/apache-maven-3.9.6/bin:$PATH"
    }
    stages{
        stage("Build") {
            steps{
                sh 'mvn clean deploy -Dmaven.test.skip=true'
            }
        }

        stage('SonarQube analysis') {
           environment  { 
            scannerHome = tool 'sonar-scanner';
           }
            steps{
                withSonarQubeEnv('sonarqube-server') { 
                    sh "${scannerHome}/bin/sonar-scanner"
                }
                
            }
        }
    }
}