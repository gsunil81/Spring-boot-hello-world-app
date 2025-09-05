pipeline {
    agent any

    tools {
        maven 'maven3'
    }

    triggers {
        githubPush()
    }

    environment {
        DOCKER_IMAGE = "sunil8179/springboot-app:v${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = "docker-sunil"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'sunil_git-cred',
                    url: 'https://github.com/gsunil81/Spring-boot-hello-world-app.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE -f Dockerfile ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Pull Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker pull $DOCKER_IMAGE
                        docker run -d --name springboot-app -p 8080:8080 $DOCKER_IMAGE
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Spring Boot app pulled and running as Docker container!"
        }
        failure {
            echo "❌ Pipeline failed. Please check the logs."
        }
        cleanup {
            cleanWs()
        }
    }
}
