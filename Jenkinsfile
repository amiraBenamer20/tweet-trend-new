def registry = 'https://code01.jfrog.io/'
def imageName = 'code01.jfrog.io/code-ttrend-docker-local/ttrend'
def version   = '2.1.2'
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
                sh 'mvn clean deploy -Dmaven.test.skip=true'
            }
        }
        
        stage('Test'){
        steps{
            echo "------Unit test start-----"
            sh 'mvn surefire-report:report'
            echo "------Unit test completed-----"
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
      }//finish analysis
      
      stage("Quality Gate"){
       steps{
        script{
          timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
            def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
            if (qg.status != 'OK') {
              error "Pipeline aborted due to quality gate failure: ${qg.status}"
            }
           }
         }
        }
       }
       
       //Publish JAR to JFog artifactory
        stage("Jar Publish") {
        steps {
            script {
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"JFrogCred"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "libs-release-local/{1}",
                              "flat": "false",
                              "props" : "${properties}",
                              "exclusions": [ "*.sha1", "*.md5"]
                            }
                         ]
                     }"""
                     def buildInfo = server.upload(uploadSpec)
                     buildInfo.env.collect()
                     server.publishBuildInfo(buildInfo)
                     echo '<--------------- Jar Publish Ended --------------->'  
            
                }
            }   
        }

           
            stage(" Docker Build ") {
              steps {
                script {
                   echo '<--------------- Docker Build Started --------------->'
                   app = docker.build(imageName+":"+version)
                   echo '<--------------- Docker Build Ends --------------->'
                }
              }
            }

                    stage (" Docker Publish "){
                steps {
                    script {
                       echo '<--------------- Docker Publish Started --------------->'  
                        docker.withRegistry(registry, 'artifactory_token'){
                            app.push()
                        }    
                       echo '<--------------- Docker Publish Ended --------------->'  
                    }
                }
            }
    }
    
}
