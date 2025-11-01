pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKER_HUB_REPO = 'gowthamkumar040319/bluegreen-demo'
        KUBE_SERVICE = 'demo-service'
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
                    // Detect which version is live (blue or green)
                    def currentColor = sh(returnStdout: true, script: "kubectl get svc ${KUBE_SERVICE} -o=jsonpath='{.spec.selector.version}'").trim()
                    def targetColor = (currentColor == 'blue') ? 'green' : 'blue'

                    echo "Currently live: ${currentColor}. Deploying to ${targetColor}..."

                    // Apply the target deployment YAML
                    sh "kubectl apply -f ${targetColor}-deployment.yaml"

                    // Update image in target deployment
                    sh "kubectl set image deployment/${targetColor}-deployment demo=${DOCKER_HUB_REPO}:latest"

                    // Wait for rollout success
                    sh "kubectl rollout status deployment/${targetColor}-deployment"

                    // Switch the service to target version
                    sh "kubectl patch svc ${KUBE_SERVICE} -p '{\"spec\": {\"selector\": {\"app\": \"demo\", \"version\": \"${targetColor}\"}}}'"

                    echo "✅ Switched live traffic to ${targetColor}."
                }
            }
        }
    }

    post {
        failure {
            echo "❌ Deployment failed. Rolling back..."
            script {
                def rollbackColor = sh(returnStdout: true, script: "kubectl get svc ${KUBE_SERVICE} -o=jsonpath='{.spec.selector.version}'").trim()
                sh "kubectl rollout undo deployment/${rollbackColor}-deployment || true"
            }
        }
        success {
            echo "🎉 Blue-Green deployment completed successfully!"
        }
    }
}
