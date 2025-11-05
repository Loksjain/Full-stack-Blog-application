pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REGISTRY = "083400432136.dkr.ecr.us-east-1.amazonaws.com"
        IMAGE_NAME = "devops-blog-app"
        K8S_NAMESPACE = "devops"
        DEPLOYMENT_NAME = "devops-blog-deployment"
        CONTAINER_NAME = "devops-blog-app"
        PATH = "/usr/local/bin:/usr/bin:/bin:/usr/local/aws-cli/v2/current/bin"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', credentialsId: 'github-creds', url: 'https://github.com/Loksjain/Full-stack-Blog-application.git'
            }
        }

        stage('Build JAR') {
            steps {
                sh 'chmod +x mvnw'
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
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} \
                        | docker login --username AWS --password-stdin ${ECR_REGISTRY}
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
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']]) {
                    sh """
                        aws eks update-kubeconfig --region ${AWS_REGION} --name devops-cluster
                        kubectl set image deployment/${DEPLOYMENT_NAME} ${CONTAINER_NAME}=${ECR_REGISTRY}/${IMAGE_NAME}:latest -n ${K8S_NAMESPACE}
                        kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${K8S_NAMESPACE}
                    """
                }
            }
        }
    }
}
