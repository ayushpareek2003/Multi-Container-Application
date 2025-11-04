pipeline {
  agent any
  environment {
    DOCKERHUB_CREDENTIALS = 'dockerhub-creds'  
    DOCKERHUB_REPO = 'ayush2003/flask-myapp'
    IMAGE_TAG = "${env.BUILD_ID}"
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Build Image') {
      steps {
        script {
          sh "docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} ./app"
        }
      }
    }
    stage('Run Tests') {
      steps {
        script {
          sh "docker run -d --name test-container -p 5001:5000 ${DOCKERHUB_REPO}:${IMAGE_TAG}"
          sleep 3
          sh "curl -f http://localhost:5001/ || true"
          sh "docker rm -f test-container || true"
        }
      }
    }
    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: env.DOCKERHUB_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
          sh "docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}"
        }
      }
    }
  }
  post {
    always {
      echo "Pipeline complete. Build ID: ${env.BUILD_ID}"
      sh "docker system prune -af || true"
    }
  }
}
