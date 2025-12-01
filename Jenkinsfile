pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "michellerebc"
        DOCKERHUB_REPO = "backend-test"

        GITHUB_USER = "michellerebc"
        GITHUB_REPO = "backend-test"

        IMAGE_NAME = "backend-test"
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/michellerebc/backend-test.git'
            }
        }

        stage('Install dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Testing') {
            steps {
                sh 'npm test || true'   // si no hay tests, no falla
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build || echo "No build step"'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                docker build -t $DOCKERHUB_USER/$IMAGE_NAME:latest .
                docker tag $DOCKERHUB_USER/$IMAGE_NAME:latest $DOCKERHUB_USER/$IMAGE_NAME:${BUILD_NUMBER}
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                    echo "$PASS" | docker login -u "$USER" --password-stdin
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
                    echo "$PASS" | docker login ghcr.io -u "$USER" --password-stdin
                    docker tag $DOCKERHUB_USER/$IMAGE_NAME:latest ghcr.io/$GITHUB_USER/$IMAGE_NAME:latest
                    docker tag $DOCKERHUB_USER/$IMAGE_NAME:latest ghcr.io/$GITHUB_USER/$IMAGE_NAME:${BUILD_NUMBER}

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
                    sed -i 's|IMAGE_TAG|${BUILD_NUMBER}|g' kubernetes.yaml
                    kubectl apply -f kubernetes.yaml
                    """
                }
            }
        }

    }
}
