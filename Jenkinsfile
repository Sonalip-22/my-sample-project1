pipeline {
    agent any

    environment {
        IMAGE_NAME = "sonalipawar22/simple-maven-demo:latest"
    }

    stages {
        stage('Cleanup workspace') {
            steps {
                echo "🧹 Cleaning workspace..."
                deleteDir()
            }
        }

        stage('Checkout Code') {
            steps {
                echo "📥 Checking out source code..."
                git url: 'https://github.com/Sonalip-22/my-sample-project1.git', branch: 'main', credentialsId: 'github-cred'
            }
        }

        stage('Build with Maven') {
            steps {
                echo "⚙️ Building with Maven..."
                timeout(time: 30, unit: 'MINUTES') {
                    sh 'mvn -B clean package'
                }
            }
        }

        stage('Verify Docker Access') {
            steps {
                echo "🐳 Verifying Docker access..."
                sh 'docker ps'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "🏗️ Building Docker image..."
                    sh '''
                        export DOCKER_BUILDKIT=0
                        docker build -t ${IMAGE_NAME} .
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        echo "🔑 Logging in to Docker Hub..."
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        echo "🚀 Pushing Docker image..."
                        sh 'docker push ${IMAGE_NAME}'
                        sh 'docker logout'
                    }
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    sh '''#!/bin/bash
                        set -e
                        echo "🚀 Starting container deployment..."

                        # Clean up any old container
                        docker rm -f simple-demo || true

                        # Detect if port 8080 is busy
                        if ss -tuln | grep -q ":8080 "; then
                          echo "⚠️ Port 8080 is in use, searching for an available port..."
                          for p in $(seq 8081 8100); do
                            if ! ss -tuln | grep -q ":$p "; then
                              PORT=$p
                              echo "✅ Found free port: $PORT"
                              break
                            fi
                          done
                        else
                          PORT=8080
                        fi

                        # Run the container on the available port
                        echo "🚢 Running container on port $PORT..."
                        docker run -d -p ${PORT}:8080 --name simple-demo ${IMAGE_NAME}

                        sleep 5
                        echo "🌐 Checking container status..."
                        docker ps | grep simple-demo || (echo "❌ Container failed to start!" && exit 1)
                        echo "✅ Container is running successfully on port $PORT!"
                        echo "🌍 Access your app at http://$(hostname -I | awk '{print $1}'):${PORT}"
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "🧼 Cleaning up running containers..."
            sh 'docker rm -f simple-demo || true'
        }
        failure {
            echo "❌ Pipeline failed. Please check the logs."
        }
    }
}
