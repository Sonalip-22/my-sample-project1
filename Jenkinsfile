pipeline {
    agent any

    environment {
        GIT_CREDENTIAL_ID = 'github-cred'
        DOCKERHUB_CREDENTIAL_ID = 'dockerhub-cred'
        IMAGE_NAME = 'sonalipawar22/simple-maven-demo:latest'
        CONTAINER_NAME = 'simple-demo'
        HOST_PORT = '8081'
        CONTAINER_PORT = '8080'
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
                    sh 'mvn -B clean package -DskipTests'
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
                    withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIAL_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        echo "🔑 Logging in to Docker Hub..."
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            echo "🚀 Pushing Docker image..."
                            docker push ${IMAGE_NAME}
                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    echo "🚀 Starting container deployment on port ${HOST_PORT}..."

                    sh '''
                        # Remove any old container safely
                        if [ "$(docker ps -aq -f name=${CONTAINER_NAME})" ]; then
                            echo "🧹 Removing old container..."
                            docker rm -f ${CONTAINER_NAME} || true
                        fi

                        # Free up host port if occupied
                        if sudo lsof -t -i:${HOST_PORT} > /dev/null; then
                            echo "⚠️ Port ${HOST_PORT} in use — killing process..."
                            sudo kill -9 $(sudo lsof -t -i:${HOST_PORT})
                        fi

                        # Run new container
                        echo "🚀 Running new container on port ${HOST_PORT}..."
                        docker run -d -p ${HOST_PORT}:${CONTAINER_PORT} --name ${CONTAINER_NAME} ${IMAGE_NAME}
                        sleep 5
                        docker ps
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "🧼 Cleaning up running containers..."
            sh 'docker rm -f ${CONTAINER_NAME} || true'
        }
        failure {
            echo "❌ Pipeline failed. Please check the logs."
        }
        success {
            echo "✅ Deployment successful! Application is running on port ${HOST_PORT}."
        }
    }
}
