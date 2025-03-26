pipeline {
    agent any

    environment {
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Pull Docker Image from Docker Hub') {
            steps {
                sh 'docker pull leonardth25/login-html-app:latest'
            }
        }

        stage('Stop Existing Container on Port 8085') {
            steps {
                sh '''
                CONTAINER_ID=$(docker ps -q --filter "publish=8085")
                if [ ! -z "$CONTAINER_ID" ]; then
                    docker stop $CONTAINER_ID
                    docker rm $CONTAINER_ID
                fi
                '''
            }
        }

        stage('Deploy to Kubernetes with Helm') {
            steps {
                sh 'helm upgrade --install login-html ./k8s'
            }
        }

        stage('Done') {
            steps {
                echo '✅ Deployed via Helm from Jenkins!'
            }
        }
    }
}
