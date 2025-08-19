pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "geniusdeck-backend:latest"
        FRONTEND_IMAGE = "geniusdeck-frontend:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Cloning repository..."
                checkout scm
            }
        }

        sstage('Build Backend') {
    steps {
        echo "Building Spring Boot backend..."
        dir('generator') {
            sh 'chmod +x mvnw'              // rend mvnw exécutable
            sh './mvnw clean package -DskipTests'
        }
    }
}



        stage('Build Frontend') {
            steps {
                echo "Building Angular frontend..."
                dir('Generator(Angular)') {
                    sh 'npm install'
                    sh 'npm run build -- --output-path=dist'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                echo "Building Docker images..."
                sh "docker build -t ${BACKEND_IMAGE} ./Generator"
                sh "docker build -t ${FRONTEND_IMAGE} ./Generator(Angular)"
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                echo "Deploying with docker-compose..."
                sh "docker-compose down"
                sh "docker-compose up -d"
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
    }
}
