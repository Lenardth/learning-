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

        stage('Run Container') {
            steps {
                script {
                    def app = docker.image('leonardth25/login-html-app:latest')
                    app.run('-d -p 8085:80')
                }
            }
        }

        stage('Done') {
            steps {
                echo '✅ Jenkins deployed Docker container from GitHub!'
            }
        }
    }
}
