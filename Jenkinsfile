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
        DOCKER_CREDENTIALS_ID = 'docker-sunil'          // DockerHub credentials
        KUBECONFIG_CREDENTIALS_ID = 'eks-kubeconfig'    // kubeconfig file credential
        NAMESPACE = 'sunil'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'sunil_git-cred',
                    url: 'https://github.com/gsunil81/Spring-boot-hello-world-app.git'
                sh 'ls -la'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE ."
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

        stage('Deploy to EKS') {
            steps {
                withCredentials([file(credentialsId: KUBECONFIG_CREDENTIALS_ID, variable: 'KUBECONFIG')]) {
                    sh '''
                        export KUBECONFIG=$KUBECONFIG
                        kubectl get namespace $NAMESPACE || kubectl create namespace $NAMESPACE
                        kubectl apply -f K8S/deployment.yaml
                        kubectl apply -f K8S/service.yaml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment to EKS namespace '$NAMESPACE' successful!"
        }
        failure {
            echo "❌ Deployment failed. Please check the logs."
        }
    }
}
