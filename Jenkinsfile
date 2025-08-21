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

        stage('Build Backend') {
            steps {
                echo "Building Spring Boot backend..."
                dir('generator') {
                    sh 'chmod +x mvnw'              
                    sh './mvnw clean package -DskipTests'
                }
            }
        }

        stage('Build Frontend') {
            steps {
                echo "Building Angular frontend..."
                dir('Generator-Angular') {
                    sh 'npm install --legacy-peer-deps'
                    sh 'npm run build -- --output-path=dist'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                        withSonarQubeEnv('SonarQube') {
                            sh """
                                ./generator/mvnw sonar:sonar \
                                  -f generator/pom.xml \
                                  -Dsonar.projectKey=geniusdeck-backend \
                                  -Dsonar.host.url=$SONAR_HOST_URL \
                                  -Dsonar.login=$SONAR_TOKEN
                            """
                        }
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                echo "Building Docker images..."
                sh "docker build -t ${BACKEND_IMAGE} ./generator"
                sh "docker build -t ${FRONTEND_IMAGE} ./Generator-Angular"
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
