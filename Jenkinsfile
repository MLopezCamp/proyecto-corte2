pipeline {
  agent any

  environment {
    REGISTRY = "docker.io"                      
    IMAGE_NAME = "ragomez333/saleor"  
    DOCKER_CREDENTIALS_ID = "docker-hub-credentials"                                     
  }

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '20'))
    timeout(time: 45, unit: 'MINUTES')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        script {
          env.TAG = sh(returnStdout: true, script: "git rev-parse --short=8 HEAD").trim()
          echo "Building tag: ${env.TAG}"
        }
      }
    }

    stage('Build image') {
      steps {
        script {
          def fullImage = "${REGISTRY}/${IMAGE_NAME}:${TAG}"
          sh "docker build -t ${fullImage} ."
        }
      }
    }

    stage('Login and Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          script {
            def fullImage = "${REGISTRY}/${IMAGE_NAME}:${TAG}"
            sh '''
              echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin ${REGISTRY}
              docker push ${fullImage}
              docker logout ${REGISTRY} || true
            '''
          }
        }
      }
    }
  }

  post {
    success {
      echo "Image pushed: ${REGISTRY}/${IMAGE_NAME}:${TAG}"
    }
    failure {
      echo "Build or push failed"
    }
    always {
      cleanWs()
    }
  }
}