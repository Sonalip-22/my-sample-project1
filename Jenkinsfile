pipeline {
    agent any

    environment {
        GIT_CREDENTIAL_ID = 'github-cred'
        DOCKERHUB_CREDENTIAL_ID = 'dockerhub-cred'
        IMAGE_NAME = 'yourdockerhubusername/simple-maven-demo'
    }git

    triggers {
        githubPush()
    }

    stages {
        stage('Pull from GitHub') {
            steps {
                git branch: 'main', url: 'https://github.com/Sonalip-22/my-sample-project1.git'
                echo "✅ Code pulled successfully."
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
                echo "✅ Maven build completed."
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:latest ."
                echo "✅ Docker image built."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIAL_ID}", usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker push ${IMAGE_NAME}:latest
                        docker logout
                    '''
                }
                echo "✅ Image pushed to DockerHub."
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    docker rm -f simple-maven-demo || true
                    docker run -d --name simple-maven-demo -p 8080:8080 ${IMAGE_NAME}:latest
                '''
                echo "✅ Container running successfully."
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline executed successfully!"
        }
        failure {
            echo "❌ Pipeline failed, check logs."
        }
    }
}