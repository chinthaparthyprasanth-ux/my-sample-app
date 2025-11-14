pipeline {
    agent any

    environment {
        AWS_REGION     = "us-east-1"
        AWS_ACCOUNT_ID = "194526776035"
        ECR_REPO       = "my-sample-app"
        IMAGE_TAG      = "${BUILD_NUMBER}"
        ECR_URL        = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Login to ECR') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    bat """
                    aws ecr get-login-password --region %AWS_REGION% ^
                    | docker login --username AWS --password-stdin %ECR_URL%
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                bat """
                docker build -t %ECR_URL%/%ECR_REPO%:%IMAGE_TAG% .
                """
            }
        }

        stage('Push to ECR') {
            steps {
                bat """
                docker push %ECR_URL%/%ECR_REPO%:%IMAGE_TAG%
                """
            }
        }
    }

    post {
        success {
            echo "Successfully built and pushed image to ECR."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
