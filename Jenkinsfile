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

        stage('Configure AWS & Kubeconfig') {
            steps {
                withCredentials([
                    string(credentialsId: AWS_ACCESS_KEY_ID_CRED, variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: AWS_SECRET_ACCESS_KEY_CRED, variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set default.region $AWS_REGION
                        aws eks --region $AWS_REGION update-kubeconfig --name $EKS_CLUSTER
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    echo "🔧 Ensuring namespace '$NAMESPACE' exists..."
                    kubectl get namespace $NAMESPACE || kubectl create namespace $NAMESPACE

                    echo "📦 Replacing image placeholder in deployment.yaml..."
                    sed -i "s@<IMAGE_PLACEHOLDER>@$DOCKER_IMAGE@g" k8s/deployment.yaml

                    echo "🚀 Applying deployment and service manifests..."
                    kubectl apply -n $NAMESPACE -f k8s/deployment.yaml
                    kubectl apply -n $NAMESPACE -f k8s/service.yaml
                '''
            }
        }

        stage('Deploy Ingress') {
            steps {
                sh '''
                    echo "🌐 Applying ingress manifest..."
                    kubectl apply -n $NAMESPACE -f k8s/ingress.yaml
                '''
            }
        }

        stage('Verify Ingress') {
            steps {
                sh '''
                    echo "🔍 Ingress DNS:"
                    kubectl get ingress springboot-app-ingress -n $NAMESPACE -o jsonpath="{.status.loadBalancer.ingress[0].hostname}"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Spring Boot app deployed successfully to EKS namespace '$NAMESPACE'!"
        }
        failure {
            echo "❌ Deployment failed. Please check the logs."
        }
        cleanup {
            cleanWs()
        }
    }
}
