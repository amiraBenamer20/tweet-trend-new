pipeline {
    agent {
        node {
            label "build-slave"
        }
    }

environement {
    PATH = "/opt/apache-maven-3.9.5/bin:$PATH"
}
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean deploy'
            }
        }
    }
}
