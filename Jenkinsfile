pipeline {
    agent any

    environment {
        GIT_CREDENTIAL_ID       = 'github-cred'
        DOCKERHUB_CREDENTIAL_ID = 'dockerhub-cred'
        IMAGE_NAME              = 'sonalipawar22/simple-maven-demo'
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
                sh 'mvn -B clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    export DOCKER_BUILDKIT=1
                    docker build -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIAL_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''#!/bin/bash
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push ${IMAGE_NAME}:latest
                    docker logout
                    '''
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    sh '''#!/bin/bash
                    set -e
                    echo "🚀 Starting container..."

                    docker rm -f simple-demo || true

                    # find a free host port between 8080–8090
                    for port in $(seq 8080 8090); do
                        if ! ss -ltn | grep -q ":$port "; then
                            FREE_PORT=$port
                            break
                        fi
                    done

                    if [ -z "$FREE_PORT" ]; then
                        echo "❌ No free port found in range 8080–8090!"
                        exit 1
                    fi

                    echo "✅ Using port $FREE_PORT for container."
                    CID=$(docker run -d -p $FREE_PORT:8080 --name simple-demo ${IMAGE_NAME}:latest)
                    echo "Container ID: $CID"
                    echo "🌐 Waiting 8s for container to start..."
                    sleep 8

                    STATE=$(docker inspect --format='{{.State.Status}}' $CID)
                    if [ "$STATE" != "running" ]; then
                        echo "❌ Container failed to start. Fetching logs..."
                        docker logs $CID || true
                        exit 1
                    fi

                    echo "✅ Container running successfully on port $FREE_PORT"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build & deployment completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
        }
        always {
            sh 'docker rm -f simple-demo || true'
        }
    }
}