pipeline {
    agent any

    environment {
        AWS_REGION       = "us-east-1"
        AWS_ACCOUNT_ID   = "194526776035"
        ECR_REPO         = "mysample_appecr"
        CLUSTER_NAME     = "mysample-app-cluster"
        SERVICE_NAME     = "mysample-app-cluster-service"
        IMAGE_TAG        = "${env.BUILD_NUMBER}"
        AWS_CREDS        = "aws-creds"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Login to ECR') {
            steps {
                withAWS(credentials: AWS_CREDS, region: AWS_REGION) {
                    sh """
                    aws ecr get-login-password --region $AWS_REGION \
                    | docker login --username AWS \
                    --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $ECR_REPO:$IMAGE_TAG .
                docker tag $ECR_REPO:$IMAGE_TAG \
                $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                """
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                """
            }
        }

        stage('Deploy to ECS') {
            steps {
                withAWS(credentials: AWS_CREDS, region: AWS_REGION) {
                    sh """
                    aws ecs update-service \
                    --cluster $CLUSTER_NAME \
                    --service $SERVICE_NAME \
                    --force-new-deployment
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment Successful"
        }
        failure {
            echo "Deployment Failed"
        }
    }
}
