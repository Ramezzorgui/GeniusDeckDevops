pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "geniusdeck-backend:latest"
        FRONTEND_IMAGE = "geniusdeck-frontend:latest"
        NEXUS_URL = "192.168.33.10:8085"    
        NEXUS_REPO = "GeniusDeck"            
        NEXUS_CREDENTIAL_ID = "nexus"
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

        stage('Run Backend Tests') {
            steps {
                echo "Running backend JUnit tests..."
                dir('generator') {
                    sh './mvnw test'
                    junit 'target/surefire-reports/*.xml'
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

        stage('Build Docker Images') {
            steps {
                echo "Building Docker images..."
                sh "docker build -t geniusdeck-backend ./generator"
                sh "docker build -t geniusdeck-frontend ./Generator-Angular"
            }
        }

        stage('Push Docker Images to Nexus') {
            steps {
                echo "Pushing Docker images to Nexus Repository..."
                withCredentials([usernamePassword(credentialsId: "${NEXUS_CREDENTIAL_ID}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh """
                        echo "$NEXUS_PASS" | docker login ${NEXUS_URL} -u "$NEXUS_USER" --password-stdin

                        docker tag geniusdeck-backend ${NEXUS_URL}/geniusdeck-backend:latest
                        docker tag geniusdeck-frontend ${NEXUS_URL}/geniusdeck-frontend:latest

                        docker push ${NEXUS_URL}/geniusdeck-backend:latest
                        docker push ${NEXUS_URL}/geniusdeck-frontend:latest

                        docker logout ${NEXUS_URL}
                    """
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                echo "Deploying with docker-compose..."
                sh "docker-compose down || true"
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
