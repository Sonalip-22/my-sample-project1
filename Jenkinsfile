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
                        echo "🚀 Starting container deployment on port 8080..."

                        # Remove old container if exists
                        docker rm -f simple-demo || true

                        # Kill any process using port 8080 (no sudo required)
                        if ss -tuln | grep -q ":8080 "; then
                          echo "⚠️ Port 8080 is in use. Killing existing process..."
                          PID=$(lsof -ti :8080)
                          kill -9 $PID
                          echo "✅ Process on port 8080 killed."
                        fi

                        # Run container on port 8080
                        docker run -d -p 8080:8080 --name simple-demo ${IMAGE_NAME}

                        # Give Jenkins something to track so exit -1 doesn’t happen
                        echo "✅ Docker container started on port 8080"
                        sleep 3

                        # Verify container is running
                        docker ps | grep simple-demo || (echo "❌ Container failed to start!" && exit 1)
                        echo "🌍 Access your app at http://$(hostname -I | awk '{print $1}'):8080"
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
