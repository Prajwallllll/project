pipeline {
    agent any

    environment {
        IMAGE_NAME = "my-flask-app"
        REGISTRY = "prajwal773"  // Your DockerHub username
    }

    stages {
        stage('Checkout') {
            steps {
                echo "📦 Pulling code from GitHub..."
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    IMAGE_TAG = "${env.BUILD_NUMBER}"
                    echo "🐳 Building Docker image ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ..."
                    sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    sh "docker rm -f flask_container_${BUILD_NUMBER} || true"
                    sh "docker run -d -p 5000:5000 --name flask_container_${BUILD_NUMBER} ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Smoke Test') {
            steps {
                script {
                    sh 'curl --retry 5 --retry-delay 1 http://localhost:5000 || (echo "Smoke test failed" && exit 1)'
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh 'docker logout'
                }
            }
        }
    }

    post {
        always {
            sh "docker rm -f flask_container_${BUILD_NUMBER} || true"
        }
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
