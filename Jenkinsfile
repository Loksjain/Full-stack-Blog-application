pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REGISTRY = "083400432136.dkr.ecr.us-east-1.amazonaws.com"
        IMAGE_NAME = "devops-blog-app"
        K8S_NAMESPACE = "devops"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', credentialsId: 'github-creds', url: 'https://github.com/Loksjain/Full-stack-Blog-application.git'
            }
        }

        stage('Build JAR') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:latest .
                    docker tag ${IMAGE_NAME}:latest ${ECR_REGISTRY}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh """
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set default.region ${AWS_REGION}
                        
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    """
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh """
                    docker push ${ECR_REGISTRY}/${IMAGE_NAME}:latest
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                    aws eks update-kubeconfig --region ${AWS_REGION} --name devops-cluster
                    kubectl set image deployment/devops-blog-deployment devops-blog-app=${ECR_REGISTRY}/${IMAGE_NAME}:latest -n ${K8S_NAMESPACE}
                """
            }
        }
    }
}
