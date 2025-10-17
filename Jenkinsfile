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
                deleteDir()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main', credentialsId: "${GIT_CREDENTIAL_ID}", url: 'https://github.com/Sonalip-22/my-sample-project1.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh "${MVN_CMD}"
            }
        }

        stage('Verify Docker Access') {
            steps {
                sh 'docker ps'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                        export DOCKER_BUILDKIT=1
                        docker build -t ${IMAGE_NAME}:latest .
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Safe way: avoid Groovy string interpolation (no more warning)
                    withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIAL_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''#!/bin/bash
                        set -e
                        echo "🔑 Logging in to Docker Hub..."
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        echo "🚀 Pushing Docker image ${IMAGE_NAME}:latest..."
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
                    sh '''
                        docker rm -f simple-demo || true
                        docker run -d -p 8080:8080 --name simple-demo ${IMAGE_NAME}:latest
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
            sh 'docker rm -f simple-demo || true'
        }
    }
}