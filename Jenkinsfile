pipeline {
    agent any

    environment {
        IMAGE_NAME = "my-flask-app"          // Change this to your app name
        REGISTRY = "prajwal773" // Change this to your DockerHub username
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
                    IMAGE_TAG = "${env.BUILD_NUMBER}" // Jenkins build number as tag
                    echo "🐳 Building Docker image ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ..."
                    sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Run Container') {
            steps {
                script {
                    echo "⚙️ Running container..."
                    sh "docker rm -f flask_container_${BUILD_NUMBER} || true"
                    sh "docker run -d -p 5000:5000 --name flask_container_${BUILD_NUMBER} ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Smoke Test') {
            steps {
                script {
                    echo "🔍 Testing if app is running..."
                    sh 'curl --retry 5 --retry-delay 1 http://localhost:5000 || (echo "Smoke test failed" && exit 1)'
                }
            }
        }

        stage('Push to DockerHub') {
            when { expression { return true } } // Optional: toggle with params
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    echo "☁️ Pushing image to DockerHub..."
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh 'docker logout'
                }
            }
        }
    }

    post {
        always {
            echo "🧹 Cleaning up containers..."
            sh "docker rm -f flask_container_${BUILD_NUMBER} || true"
        }
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs!"
        }
    }
}
