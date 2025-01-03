pipeline {
    agent any
    
    tools {
        go 'Go 1.22' // Ensure Go is configured in Jenkins global tool configuration
    }
    environment {
        DOCKER_CREDENTIALS_ID ='docker-red' // Jenkins credentials for DockerHub
        DOCKERHUB_USERNAME = 'labi007'
        GIT_CREDENTIALS_ID = 'git' // Jenkins credentials for Git
        GO_APP_NAME = 'my-go-app'
        DOCKER_IMAGE = "${DOCKERHUB_USERNAME}/${GO_APP_NAME}:${BUILD_NUMBER}"
        REGISTRY_CREDENTIALS = 'docker-red' // Replace with your Jenkins credential ID
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
        /*
        stage('Docker Build and Push') {
            steps {
                script {
                  // sh ' docker build -t ${DOCKER_IMAGE} .'
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                       // def dockerImage = docker.build("${DOCKERHUB_USERNAME}/${GO_APP_NAME}:${env.BUILD_NUMBER}")
                        def dockerImage = docker.build("${DOCKER_IMAGE}")

                        dockerImage.push()
                    }
                }
            }
        }
        */

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }
         stage('Push') {
            steps {
                script {
                  // sh ' docker build -t ${DOCKER_IMAGE} .'
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-red') {
                       // def dockerImage = docker.build("${DOCKERHUB_USERNAME}/${GO_APP_NAME}:${env.BUILD_NUMBER}")
                        //def dockerImage = docker.build("${DOCKER_IMAGE}")
                     def dockerImage = docker.image("${DOCKER_IMAGE}")
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Update Helm Chart Tag') {
            environment {
                GIT_REPO_NAME = "go-web-app-devops"
                GIT_USER_NAME = "Afolabigroup"
            }
            steps {
                script {
                    sh '''
                    git config user.email "Afolabiewuola@gmail.com"
                    git config user.name "Ewuola1"
                    sed -i "s/tag: .*/tag: \\"${BUILD_NUMBER}\\"/" helm/go-web-app-chart/values.yaml
                    git add helm/go-web-app-chart/values.yaml
                    git commit -m "Update tag in Helm chart"
                    git push
                    '''
                }
            }
        }
        
        
    }
    
    
}
