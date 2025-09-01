pipeline {
    agent any

    triggers {
        githubPush()
    }

    tools {
        maven 'maven3'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'local-jenkins-sunil',
                    url: 'https://github.com/gsunil81/Spring-boot-hello-world-app.git'
                sh 'ls -la'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQube') {   // <-- Name must match Jenkins UI config
                    sh 'mvn sonar:sonar -Dsonar.projectKey=springboot-demo'
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                nohup java -jar target/simple-hello-sunil-1.0.0.jar --server.port=9090 > app.log 2>&1 &
                '''
            }
        }
    }
}
