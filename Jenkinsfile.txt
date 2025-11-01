pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'debi1997/testimage'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "Checking out code from GitHub..."
                git branch: 'master', url: 'https://github.com/dev14368/Jenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                bat "docker build -t %DOCKER_HUB_REPO%:${BUILD_NUMBER} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "Pushing Docker image to Docker Hub..."
                withCredentials([string(credentialsId: 'dockerhub-token', variable: 'DOCKER_HUB_PASS')]) {
                    bat '''
                        echo %DOCKER_HUB_PASS% | docker login -u your-dockerhub-username --password-stdin
                        docker push %DOCKER_HUB_REPO%:${BUILD_NUMBER}
                        docker tag %DOCKER_HUB_REPO%:${BUILD_NUMBER} %DOCKER_HUB_REPO%:latest
                        docker push %DOCKER_HUB_REPO%:latest
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Docker image built and pushed successfully!'
        }
        failure {
            echo '❌ Build failed! Please check logs.'
        }
    }
}
