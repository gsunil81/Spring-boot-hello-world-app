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
        NAMESPACE = "sunil"
        HELM_RELEASE_NAME = "springboot-app"
        HELM_CHART_PATH = "./helm-chart"
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

        stage('Deploy via Helm') {
            steps {
                sh '''
                    echo "🔧 Ensuring namespace '$NAMESPACE' exists..."
                    kubectl get namespace $NAMESPACE || kubectl create namespace $NAMESPACE

                    echo "🚀 Deploying Helm chart to local Kubernetes..."
                    helm upgrade --install $HELM_RELEASE_NAME $HELM_CHART_PATH \
                        --namespace $NAMESPACE \
                        --set image.repository=${DOCKER_IMAGE%:*} \
                        --set image.tag=${DOCKER_IMAGE##*:}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "🔍 Helm release status:"
                    helm status $HELM_RELEASE_NAME --namespace $NAMESPACE

                    echo "🔍 Service details:"
                    kubectl get svc -n $NAMESPACE

                    echo "🔍 Pod status:"
                    kubectl get pods -n $NAMESPACE
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Spring Boot app deployed successfully via Helm to local Kubernetes namespace '$NAMESPACE'!"
        }
        failure {
            echo "❌ Deployment failed. Please check the logs."
        }
        cleanup {
            cleanWs()
        }
    }
}
