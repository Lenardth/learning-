pipeline {
    agent any

    environment {
        // More reliable KUBECONFIG path that works across systems
        KUBECONFIG = "${env.HOME}/.kube/config"
        // Add Helm timeout to prevent hanging
        HELM_TIMEOUT = "5m"
    }

    stages {
        stage('Verify Environment') {
            steps {
                script {
                    // Verify Docker is available
                    sh 'docker --version'
                    // Verify Helm is installed
                    sh 'helm version'
                    // Verify kubectl access
                    sh 'kubectl cluster-info'
                }
            }
        }

        stage('Pull Docker Image') {
            steps {
                sh 'docker pull leonardth25/login-html-app:latest'
            }
        }

        stage('Cleanup Previous Deployment') {
            steps {
                script {
                    // More robust container cleanup
                    def runningContainers = sh(script: 'docker ps -q --filter "publish=8085"', returnStdout: true).trim()
                    if (runningContainers) {
                        sh """
                        docker stop ${runningContainers}
                        docker rm ${runningContainers}
                        """
                    }
                    
                    // Also clean up any dangling containers
                    sh 'docker system prune -f'
                }
            }
        }

        stage('Helm Deployment') {
            steps {
                script {
                    // More robust Helm deployment with checks
                    try {
                        sh """
                        helm upgrade --install login-html ./k8s \
                            --timeout ${HELM_TIMEOUT} \
                            --atomic \
                            --cleanup-on-fail
                        """
                    } catch (err) {
                        // Get deployment status if failed
                        sh 'helm status login-html || true'
                        sh 'kubectl get pods -o wide'
                        error("Helm deployment failed: ${err}")
                    }
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    // Verify pods are running
                    sh 'kubectl wait --for=condition=ready pod -l app=login-html --timeout=300s'
                    // Verify service is available
                    sh 'kubectl get svc login-html'
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed - cleanup resources"
            sh 'docker system prune -f'
        }
        success {
            echo '✅ Deployment successful!'
            // Optional: Notify success (Slack, email, etc.)
        }
        failure {
            echo '❌ Deployment failed!'
            // Optional: Notify failure
        }
    }
}