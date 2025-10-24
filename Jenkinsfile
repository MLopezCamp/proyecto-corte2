pipeline {
    agent any

    environment {
        REGISTRY = 'docker.io'
        IMAGE_NAME = 'ragomez333/saleor'
        COMPOSE_FILE = '.devcontainer/docker-compose.yml'
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

                        // Etiqueta como latest
                        sh "docker tag ${IMAGE_TAG} ${REGISTRY}/${IMAGE_NAME}:latest"
                        sh "docker push ${REGISTRY}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }

       stage('Deploy containers') {
    steps {
        script {
            // Detener y eliminar contenedores previos
            sh "docker compose -f ${COMPOSE_FILE} down -v || true"

            // Descargar e iniciar los servicios definidos en docker-compose.yml
            sh "docker compose -f ${COMPOSE_FILE} pull"
            sh "docker compose -f ${COMPOSE_FILE} up -d"

            // Esperar a que la base de datos esté lista
            echo "Esperando que la base de datos esté lista..."
            sh "sleep 15"

            // Ejecutar migraciones
            echo "Ejecutando migraciones..."
            sh "docker exec saleor python3 /app/saleor/saleor/app/manage.py migrate"

            // Crear superusuario automáticamente
            echo "Creando superusuario..."
            sh '''
            docker exec saleor python3 /app/saleor/saleor/app/manage.py shell -c "from django.contrib.auth import get_user_model; User = get_user_model(); User.objects.filter(email='admin@saleor.io').exists() or User.objects.create_superuser('admin@saleor.io', 'admin1234'); print('Superusuario creado correctamente')"
            '''

            // Poblar base de datos
            echo "Poblando base de datos..."
            sh "docker exec saleor python3 /app/saleor/saleor/app/manage.py populatedb --createsuperuser"
        }
    }
}

    }

    post {
        always {
            echo "Limpiando workspace..."
            cleanWs()
        }
        failure {
            echo 'Build o despliegue falló'
        }
        success {
            echo 'Pipeline completado correctamente: contenedores listos, migraciones y datos cargados'
        }
    }
}
