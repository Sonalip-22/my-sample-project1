pipeline {
  agent any

  environment {
    GIT_CREDENTIAL_ID        = 'github-cred'
    DOCKERHUB_CREDENTIAL_ID  = 'dockerhub-cred'
    IMAGE_NAME               = 'sonalipawar22/simple-maven-demo'
    MVN_CMD                  = 'mvn -B -DskipTests=false'   // change if you want to skip tests
  }

  stages {

    stage('Prepare / Debug info') {
      steps {
        // lightweight debug output to help troubleshooting if something breaks
        sh '''
          echo "===== Agent info ====="
          uname -a
          whoami || true
          pwd
          echo "===== Workspace listing ====="
          ls -la || true
        '''
      }
    }

    stage('Cleanup workspace') {
      steps {
        // safer to start with a clean workspace so leftover root-owned files don't break mvn clean
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
        sh "${MVN_CMD} clean package"
      }
    }

    stage('Verify Docker & Network') {
      steps {
        // quick network + docker pre-checks so any networking problem fails fast with a clear message
        sh '''
          echo "Checking DNS for Docker registry..."
          nslookup registry-1.docker.io 8.8.8.8 || (echo "DNS lookup failed" && exit 1)

          echo "Checking registry connectivity with curl..."
          curl -sSfI https://registry-1.docker.io/v2/ || (echo "Cannot reach registry-1.docker.io" && exit 1)

          echo "Checking docker daemon and pull test image..."
          docker ps || (echo "Docker daemon not accessible by this user" && exit 1)
          docker pull --quiet hello-world || (echo "Docker pull failed" && exit 1)
        '''
      }
    }

    stage('Build Docker Image') {
      steps {
        // enable BuildKit for faster, modern builds and quieter output
        // retry to tolerate short networking flakiness
        script {
          retry(2) {
            sh '''
              export DOCKER_BUILDKIT=1
              docker build -t ${IMAGE_NAME}:latest .
            '''
          }
        }
      }
    }

    stage('Push Docker Image') {
      steps {
        // Use Jenkins credentials and avoid Groovy string interpolation for secrets
        withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIAL_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          script {
            retry(2) {
              sh '''
                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                docker push ${IMAGE_NAME}:latest
              '''
            }
          }
        }
      }
    }

    stage('Run Container (smoke)') {
      steps {
        // Run container as smoke test — ok to ignore failure of run (adjust as needed)
        sh '''
          docker rm -f simple-demo || true
          docker run -d --name simple-demo -p 8080:8080 ${IMAGE_NAME}:latest || (echo "docker run failed" && exit 1)
          # small wait to allow app to start (adjust if your app needs longer)
          sleep 5
          # optional quick health-check - try to fetch root (returns non-zero on failure)
          curl -sSf http://localhost:8080/ || echo "Warning: app did not respond on / (may be fine depending on app)"
        '''
      }
    }
  }

  post {
    success {
      echo '✅ Build & deployment succeeded'
    }
    failure {
      echo '❌ Pipeline failed. Check logs above.'
    }
    always {
      // best-effort cleanup of smoke container
      sh 'docker rm -f simple-demo || true'
    }
  }
}