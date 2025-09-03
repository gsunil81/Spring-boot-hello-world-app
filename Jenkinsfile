pipeline {
    agent any

    tools {
        maven 'maven3'
    }

    triggers {
        githubPush()
    }

    environment {
        AWS_REGION = "eu-west-2"
        EKS_CLUSTER = "batch4-Team3-cluster"
        DOCKER_IMAGE = "sunil8179/springboot-app:v${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = "docker-sunil"
        AWS_ACCESS_KEY_ID_CRED = "aws-access-key-id"
        AWS_SECRET_ACCESS_KEY_CRED = "aws-secret-access-key"
        NAMESPACE = "sunil"
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
               
