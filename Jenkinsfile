pipeline {
    agent {
        node {
            label "build-slave"
        }
    }

    stages {
        stage('Clone-code') {
            steps {
                git branch: 'main', url: 'https://github.com/amiraBenamer20/tweet-trend-new'
            }
        }
    }
}
