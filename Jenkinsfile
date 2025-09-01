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
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds'
        KUBECONFIG_CREDENTIALS_ID = 'eks-kubeconfig'
        NAMESPACE = 'sunil'
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

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarQube') {
                    sh 'mvn sonar:sonar -DskipTests -Dsonar.projectKey=springboot-demo'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_IMAGE
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([file(credentialsId: KUBECONFIG_CREDENTIALS_ID, variable: 'KUBECONFIG')]) {
                    sh """
                        export KUBECONFIG=$KUBECONFIG
                        kubectl config use-context your-eks-context
                        kubectl set image deployment/springboot-deployment springboot-container=$DOCKER_IMAGE -n $NAMESPACE
                        kubectl rollout status deployment/springboot-deployment -n $NAMESPACE
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build and deployment to namespace '$NAMESPACE' successful!"
        }
        failure {
            echo "❌ Build failed. Check logs for details."
        }
        always {
            sh "docker rmi $DOCKER_IMAGE || true"
        }
    }
}
