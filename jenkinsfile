pipeline {
    agent any

    triggers {
        githubPush()
    }
    tools{
        maven 'maven'
    }
    environment {
        SONARQUBE_URL = 'https://d6312cd2eccd.ngrok-free.app/projects'
        SONARQUBE_TOKEN = credentials ('local-sonarqube')
    }
    stages {
        stage('Checkout'){
            steps{
                git branch: 'main', credentialsId: 'local-jenkins-rashi', url: 'https://github.com/Rashi3108/MySpringBootApp.git'
                sh 'ls -la'
            }
        }
        stage('Build'){
            steps{
                sh 'mvn clean install'
            }
        }
        stage('Package'){
            steps{
                sh 'mvn package'
            }
        }
        stage('SonarQube Analysis'){
            steps{
                sh '''
                mvn sonar:sonar \
                  -Dsonar.projectKey=springboot-demo \
                  -Dsonar.host.url=https://d6312cd2eccd.ngrok-free.app \
                  -Dsonar.login=$SONARQUBE_TOKEN
                '''
            }
        }
        stage('Deploy'){
            steps{
                sh '''
                nohup java -jar target/simple-hello-rashi-1.0.0.jar --server.port=9090 > app.log 2>&1 &
                '''
            }
        }
    }
}
