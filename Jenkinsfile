pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        ECR_REPO = '083400432136.dkr.ecr.us-east-1.amazonaws.com/devops-blog-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/<your-repo>'
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("devops-blog-app:latest", ".")
                }
            }
        }

        stage('Login to ECR') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    sh '''
                      aws ecr get-login-password --region us-east-1 \
                      | docker login --username AWS --password-stdin 083400432136.dkr.ecr.us-east-1.amazonaws.com
                    '''
                }
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                  docker tag devops-blog-app:latest $ECR_REPO:latest
                  docker push $ECR_REPO:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    sh '''
                      aws eks update-kubeconfig --name devops-cluster --region us-east-1
                      kubectl rollout restart deployment/devops-blog-deployment -n devops
                    '''
                }
            }
        }
    }
}
