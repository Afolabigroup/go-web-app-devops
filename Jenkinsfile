/*pipeline {
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
*/

pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-cred' // Jenkins credentials for DockerHub
        DOCKERHUB_USERNAME = 'labi007'
        GIT_CREDENTIALS_ID = 'git' // Jenkins credentials for Git
        GO_APP_NAME = 'my-go-app'
        DOCKER_IMAGE = "${DOCKERHUB_USERNAME}/${GO_APP_NAME}:${BUILD_NUMBER}"
        REGISTRY_CREDENTIALS = 'docker-cred' // Replace with your Jenkins credential ID
    }

    tools {
        go 'Go 1.22' // Ensure Go is configured in Jenkins global tool configuration
    }

    stages {
        stage('Checkout Repository') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'go build -o go-web-app'
            }
        }

        stage('Test') {
            steps {
                sh 'go test ./...'
            }
        }
        
        stage('Code Quality') {
            steps {
                sh 'golangci-lint run'
            }
        }
        
        stage('Docker Build and Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                       // def dockerImage = docker.build("${DOCKERHUB_USERNAME}/${GO_APP_NAME}:${env.BUILD_NUMBER}")
                        def dockerImage = docker.build("${DOCKER_IMAGE}")

                        dockerImage.push()
                    }
                }
            }
        }

        stage('Update Helm Chart Tag') {
            steps {
                script {
                    sh '''
                    sed -i "s/tag: .*/tag: \\"${BUILD_NUMBER}\\"/" helm/go-web-app-chart/values.yaml
                    git config user.email "your-email@example.com"
                    git config user.name "Your Name"
                    git add helm/go-web-app-chart/values.yaml
                    git commit -m "Update tag in Helm chart"
                    git push
                    '''
                }
            }
        }
    
        
    }

    post {
        always {
            cleanWs() // Clean up workspace after build
        }
    }
}
