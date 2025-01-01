pipeline {
    agent any
    tools {
        go 'Go 1.22' // Make sure to configure this Go version in "Global Tool Configuration"
    }
    stages {
        stage('Build') {
            steps {
                sh 'go build -o go-web-app'
            }
        }
    }
}

