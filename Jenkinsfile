pipeline {
    agent any

    environment {
        IMAGE_NAME = "l3shan/gitops-demo"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/L3shan-sv/gitops.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $IMAGE_NAME:${BUILD_NUMBER} app/"
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push $IMAGE_NAME:${BUILD_NUMBER}"
            }
        }

        stage('Update K8s Manifest') {
            steps {
                sh """
                sed -i 's|image:.*|image: $IMAGE_NAME:${BUILD_NUMBER}|' k8s/deployment.yaml
                git add k8s/deployment.yaml
                git commit -m "Update image to ${BUILD_NUMBER}"
                git push
                """
            }
        }
    }
}
