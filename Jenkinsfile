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

        stage("JUnit test"){
            steps{
                echo "--------unit test started----------"
                sh 'mvn surefire-report:report'
                echo "--------unit test completed----------"
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