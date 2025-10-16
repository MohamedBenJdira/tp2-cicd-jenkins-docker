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
        cid=$(docker run -d -p 8080:8080 ${IMAGE_NAME}:build-${BUILD_NUMBER})
        for i in {1..30}; do
          if curl -sf http://localhost:8080 > /dev/null; then ok=1; break; fi
          sleep 1
        done
        docker logs $cid
        docker stop $cid
        test "$ok" = "1"
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
