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
        sh '''
        cid=$(docker run -d ${IMAGE_NAME}:build-${BUILD_NUMBER})

        ok=0
        for i in $(seq 1 60); do
          if docker run --rm --network container:$cid curlimages/curl:8.10.1 -sf http://localhost:8080/ >/dev/null; then
            ok=1
            break
          fi
          echo "Waiting for app to start... attempt $i"
          sleep 2
        done

        docker logs "$cid" | tail -n 200 || true
        docker stop "$cid" >/dev/null
        test "$ok" -eq 1
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
