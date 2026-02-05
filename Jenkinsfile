pipeline {
    agent any

    environment {
        IMAGE_NAME = "yourdockerhubusername/gitops-demo"
        BRANCH = "main" // make sure this matches your repo's default branch
    }

    stages {
        stage('Checkout') {
            steps {
                // Use full Git SCM syntax to avoid branch issues
                git branch: "${BRANCH}",
                    url: "https://github.com/L3shan-sv/gitops.git"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image from app folder
                    sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} app/"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Update K8s Manifest') {
            steps {
                script {
                    sh """
                    sed -i 's|image:.*|image: ${IMAGE_NAME}:${BUILD_NUMBER}|' k8s/deployment.yaml
                    git config user.email "jenkins@example.com"
                    git config user.name "jenkins"
                    git add k8s/deployment.yaml
                    git commit -m "Update image to ${BUILD_NUMBER}"
                    git push origin ${BRANCH} || echo "No changes to push"
                    """
                }
            }
        }
    }

    post {
        failure {
            echo "Build failed! Check the logs."
        }
        success {
            echo "Build and deploy pipeline completed successfully."
        }
    }
}
