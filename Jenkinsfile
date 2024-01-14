def registry = 'https://queensland.jfrog.io/'
def imageName = 'queensland.jfrog.io/project-docker-local/ttrend'
def version   = '2.1.2'  
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

        /*stage("JUnit test"){
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

        // No need to occupy a node
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
        }*/


        stage("Jar publish"){
            steps{
                script{
                    echo '<--------------- Jar Publish Started --------------->'
                     def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"JFrog-Cred"
                     def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                     def uploadSpec = """{
                          "files": [
                            {
                              "pattern": "jarstaging/(*)",
                              "target": "project-libs-release-local/{1}",
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

        stage("Build image from jar"){
            steps{
                script{
                   echo '<--------------- Docker Build Started --------------->'
                   app = docker.build(imageName+":"+version)
                   echo '<--------------- Docker Build Ends --------------->'
                }
            }
        }

        stage("Publish image to JFrog"){
            steps{
                script{
                    echo '---start pushing image---'
                    docker.withRegistry(registry, 'JFrog-Cred'){
                        app.push()
                    }
                    echo '---finish pushing image---'
                }
            }
        }

        /*stage("Deploy on Kubernetes"){
            steps{
                script{
                    echo '--------------------------Start deploying------------------'
                    sh './deploy.sh'
                    echo '--------------------------------App is deployed----------------------'
                }
            }
        }*/

        stage("Deploy on Kubernetes using Helm"){
            steps{
                script{
                    echo '---------------Start deploying using Helm---------------------'
                    sh 'helm install ttrend trend-new-app-0.1.0.tgz'
                    echo '---------------Finish deploying-------------------------------'
                }
            }
        }
    }
}