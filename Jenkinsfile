pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "michelleosorio"
        DOCKERHUB_REPO = "backend-test"

        GITHUB_USER = "michellerebc"
        GITHUB_REPO = "backend-test"

        IMAGE_NAME = "backend-test"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/michellerebc/backend-test.git'
            }
        }

        stage('Install dependencies') {
            steps {
                sh '''
                echo "Instalando dependencias (npm install se ejecuta dentro del Dockerfile durante el build de la imagen)"
                ls -la
                '''
            }
        }

        stage('Testing') {
            steps {
                sh '''
                echo "Ejecutando tests (simulados para el laboratorio)"
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                echo "Ejecutando build de la aplicación (la compilación real ocurre en el docker build)"
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                echo "Construyendo imagen Docker..."
                docker build -t $DOCKERHUB_USER/$IMAGE_NAME:latest .
                docker tag $DOCKERHUB_USER/$IMAGE_NAME:latest $DOCKERHUB_USER/$IMAGE_NAME:${BUILD_NUMBER}
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                    echo "Autenticando en Docker Hub..."
                    echo "$PASS" | docker login -u "$USER" --password-stdin
                    echo "Subiendo imagen a Docker Hub..."
                    docker push $DOCKERHUB_USER/$IMAGE_NAME:latest
                    docker push $DOCKERHUB_USER/$IMAGE_NAME:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Push to GitHub Packages') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github_packages_credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                    echo "Autenticando en GHCR..."
                    echo "$PASS" | docker login ghcr.io -u "$USER" --password-stdin
                    echo "Taggeando imagen para GHCR..."
                    docker tag $DOCKERHUB_USER/$IMAGE_NAME:latest ghcr.io/$GITHUB_USER/$IMAGE_NAME:latest
                    docker tag $DOCKERHUB_USER/$IMAGE_NAME:latest ghcr.io/$GITHUB_USER/$IMAGE_NAME:${BUILD_NUMBER}
                    echo "Subiendo imagen a GHCR..."
                    docker push ghcr.io/$GITHUB_USER/$IMAGE_NAME:latest
                    docker push ghcr.io/$GITHUB_USER/$IMAGE_NAME:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                    echo "Actualizando manifiesto de Kubernetes con el build ${BUILD_NUMBER}..."
                    sed -i 's|IMAGE_TAG|${BUILD_NUMBER}|g' kubernetes.yaml || echo "Si ya estaba reemplazado, no pasa nada"
                    echo "Aplicando kubernetes.yaml al cluster..."
                    kubectl apply -f kubernetes.yaml
                    """
                }
            }
        }

    }
}
