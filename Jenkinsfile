pipeline {
    agent any

    environment {
        GIT_CREDENTIAL_ID        = 'github-cred'
        DOCKERHUB_CREDENTIAL_ID  = 'dockerhub-cred'
        IMAGE_NAME               = 'sonalipawar22/simple-maven-demo'
        MVN_CMD                  = 'mvn -B clean package'
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
                git branch: 'main', credentialsId: "${GIT_CREDENTIAL_ID}", url: 'https://github.com/Sonalip-22/my-sample-project1.git'
            }
        }

        stage('Build with Maven') {
            steps {
                echo "⚙️ Building with Maven..."
                timeout(time: 30, unit: 'MINUTES') {
                    sh "${MVN_CMD}"
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
                        docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest .
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIAL_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''#!/bin/bash
                        set -e
                        echo "🔑 Logging in to Docker Hub..."
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        echo "🚀 Pushing Docker images..."
                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${IMAGE_NAME}:latest
                        docker logout
                        '''
                    }
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    echo "🚀 Starting container..."
                    sh '''
                        docker rm -f simple-demo || true
                        docker run -d -p 8080:8080 --name simple-demo ${IMAGE_NAME}:latest
                        sleep 5
                        echo "🌐 Checking container status..."
                        docker ps | grep simple-demo || (echo "❌ Container failed to start!" && exit 1)
                        echo "✅ Container started successfully!"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build and deployment successful!'
        }
        failure {
            echo '❌ Pipeline failed. Please check the logs.'
        }
        always {
            echo '🧼 Cleaning up running containers...'
            sh 'docker rm -f simple-demo || true'
        }
    }
}
