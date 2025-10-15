pipeline {
    agent any

    environment {
        GIT_CREDENTIAL_ID = 'github-cred'
        DOCKERHUB_CREDENTIAL_ID = 'dockerhub-cred'
        IMAGE_NAME = 'sonalip22/simple-maven-demo'
    }

    stages {

        stage('Cleanup workspace') {
            steps {
                deleteDir() // prevents old files from blocking mvn clean
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

        stage('Verify Docker Access') {
            steps {
                sh 'docker ps || (echo "❌ Jenkins user cannot access Docker. Run: sudo usermod -aG docker jenkins" && exit 1)'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIAL_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Run Container') {
            steps {
                sh "docker run -d -p 8080:8080 --name simple-demo ${IMAGE_NAME}:latest || true"
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
            sh "docker rm -f simple-demo || true"
        }
    }
}