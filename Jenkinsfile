pipeline {
    agent any
    
    tools {
        go 'Go 1.22' // Ensure Go is configured in Jenkins global tool configuration
    }
    environment {
        DOCKER_CREDENTIALS_ID ='docker-cred' // Jenkins credentials for DockerHub
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
        
        stage('Build and Push Docker Image') {
            environment {
                    //DOCKER_IMAGE = "labi007/java-test-app:${BUILD_NUMBER}" // Dynamic image name with build number
                // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-app/Dockerfile" // Dockerfile location
                    REGISTRY_CREDENTIALS = credentials('docker-cred') // Jenkins credential ID for Docker registry
                }
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_IMAGE} ."
                    
                    // Create a Docker image object
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    
                    // Push the image to the Docker registry
                    docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
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
