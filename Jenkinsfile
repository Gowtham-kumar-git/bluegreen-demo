pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKER_HUB_REPO = 'gowthamkumar040319/bluegreen-demo'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Gowtham-kumar-git/bluegreen-demo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t $DOCKER_HUB_REPO:latest .
                    """
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    sh """
                    echo "$DOCKER_HUB_CREDENTIALS_PSW" | docker login -u "$DOCKER_HUB_CREDENTIALS_USR" --password-stdin
                    docker push $DOCKER_HUB_REPO:latest
                    """
                }
            }
        }

        stage('Deploy to Kubernetes (Blue/Green)') {
            steps {
                script {
                    sh """
                    kubectl apply -f blue-deployment.yaml || true
                    kubectl apply -f green-deployment.yaml || true
                    """
                }
            }
        }
    }
}
