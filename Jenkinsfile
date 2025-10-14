pipeline {
    agent any

    environment {
        GIT_CREDENTIAL_ID = 'github-cred'
        DOCKERHUB_CREDENTIAL_ID = 'dockerhub-cred'
        IMAGE_NAME = 'sonalip22/simple-maven-demo' // <-- use your real Docker Hub username
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', credentialsId: "${GIT_CREDENTIAL_ID}", url: 'https://github.com/Sonalip-22/my-sample-project1.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIAL_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker push ${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    sh "docker run -d -p 8080:8080 ${IMAGE_NAME}:latest"
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
    }
}
