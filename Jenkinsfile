pipeline {
  agent any
  environment {
    IMAGE_NAME = 'hamaaa/docker-demo'
  }
  options { timestamps() }

  stages {
    stage('Checkout') {
      steps {
        echo '📦 Checking out code...'
        checkout scm
      }
    }

    stage('Build Docker image') {
      steps {
        echo '🔨 Building Docker image...'
        sh "docker build -t ${IMAGE_NAME}:build-${env.BUILD_NUMBER} ."
      }
    }

    stage('Smoke test container') {
  steps {
      echo '🧪 Running container test...'
      sh '''
      set -e
      cid=$(docker run -d -p 8080:8080 ${IMAGE_NAME}:build-${BUILD_NUMBER})
      ok=0
      for i in {1..40}; do
        if curl -sf http://localhost:8080 > /dev/null; then
          echo "✅ App is responding!"
          ok=1
          break
        fi
        echo "Waiting for app to start..."
        sleep 2
      done
      docker logs $cid
      docker stop $cid
      if [ "$ok" -ne 1 ]; then
        echo "❌ App did not respond in time"
        exit 1
      fi
      '''
    }
  }


    stage('Push to Docker Hub') {
      steps {
        echo '🚀 Pushing image to Docker Hub...'
        withCredentials([
          string(credentialsId: 'dockerhub_user', variable: 'DOCKER_USER'),
          string(credentialsId: 'dockerhub_token', variable: 'DOCKER_TOKEN')
        ]) {
          sh 'echo $DOCKER_TOKEN | docker login -u $DOCKER_USER --password-stdin'
          sh "docker tag ${IMAGE_NAME}:build-${env.BUILD_NUMBER} ${IMAGE_NAME}:${env.BUILD_NUMBER}"
          sh "docker tag ${IMAGE_NAME}:build-${env.BUILD_NUMBER} ${IMAGE_NAME}:latest"
          sh "docker push ${IMAGE_NAME}:${env.BUILD_NUMBER}"
          sh "docker push ${IMAGE_NAME}:latest"
        }
      }
    }
  }

  post {
    always {
      echo '🧹 Cleaning workspace...'
      cleanWs()
    }
  }
}
