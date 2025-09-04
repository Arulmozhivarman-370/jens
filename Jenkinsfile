pipeline {
  agent any
  environment {
    IMAGE = "DOCKERHUB_USER/springboot-jenkins"   // <- replace DOCKERHUB_USER
    TAG   = "${env.BUILD_NUMBER}"
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build (Maven)') {
      steps {
        sh 'mvn -B -DskipTests package'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${IMAGE}:${TAG} ."
      }
    }

    stage('Login & Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
          sh 'echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin'
          sh "docker push ${IMAGE}:${TAG}"
        }
      }
    }

    stage('Deploy (run container)') {
      steps {
        sh '''
          docker rm -f springboot-jenkins-app || true
          docker run -d --name springboot-jenkins-app -p 8080:8080 ${IMAGE}:${TAG}
        '''
      }
    }
  }

  post {
    success { echo "Pipeline finished — app available on port 8080" }
    failure { echo "Pipeline failed — check console output" }
  }
}

