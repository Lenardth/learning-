pipeline {
    agent any

    stages {
        stage('Pull Docker Image from Docker Hub') {
            steps {
                script {
                    docker.image('leonardth25/login-html-app:latest').pull()
                }
            }
        }

        stage('Stop Existing Container on Port 8085') {
            steps {
                script {
                    sh '''
                    CONTAINER_ID=$(docker ps -q --filter "publish=8085")
                    if [ ! -z "$CONTAINER_ID" ]; then
                        echo "Stopping container on port 8085: $CONTAINER_ID"
                        docker stop $CONTAINER_ID
                        docker rm $CONTAINER_ID
                    fi
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'kubectl apply -f k8s/deployment.yaml --validate=false'
                    sh 'kubectl apply -f k8s/service.yaml --validate=false'
                }
            }
        }

        stage('Done') {
            steps {
                echo '✅ Jenkins pulled image and deployed to Kubernetes!'
            }
        }
    }
}
