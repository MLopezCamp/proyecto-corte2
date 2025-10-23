pipeline {
    agent any

    environment {
        REGISTRY = 'docker.io'
        IMAGE_NAME = 'ragomez333/saleor'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Get commit hash') {
            steps {
                script {
                    COMMIT = sh(returnStdout: true, script: 'git rev-parse --short=8 HEAD').trim()
                    IMAGE_TAG = "${REGISTRY}/${IMAGE_NAME}:${COMMIT}"
                    echo "Imagen a construir: ${IMAGE_TAG}"
                }
            }
        }

        stage('Build image') {
            steps {
                script {
                    sh "docker build -t ${IMAGE_TAG} ."
                }
            }
        }

        stage('Login and Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    script {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin ${REGISTRY}"
                        sh "docker push ${IMAGE_TAG}"
                        echo "Imagen subida correctamente: ${IMAGE_TAG}"
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        failure {
            echo 'Build o push fallo'
        }
        success {
            echo 'Pipeline completado correctamente'
        }
    }
}
