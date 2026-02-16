pipeline {
    agent {
        kubernetes {
            label 'docker-agent'
            defaultContainer 'docker'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:24.0.5
    command:
    - cat
    tty: true
    volumeMounts:
    - name: docker-socket
      mountPath: /var/run/docker.sock
  volumes:
  - name: docker-socket
    hostPath:
      path: /var/run/docker.sock
"""
        }
    }

    environment {
        IMAGE_NAME = "l3shan/gitops-demo"
        BRANCH = "main"
        DOCKERHUB_CREDENTIALS = "dockerhub-creds" // replace with your Jenkins creds ID
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${BRANCH}", url: "https://github.com/L3shan-sv/gitops.git"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} app/"
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Update K8s Manifest') {
            steps {
                sh """
                sed -i 's|image:.*|image: ${IMAGE_NAME}:${BUILD_NUMBER}|' k8s/deployment.yaml
                git config user.email "jenkins@example.com"
                git config user.name "jenkins"
                git add k8s/deployment.yaml
                git commit -m "Update image to ${BUILD_NUMBER}" || echo "No changes to commit"
                git push origin ${BRANCH} || echo "No changes to push"
                """
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
