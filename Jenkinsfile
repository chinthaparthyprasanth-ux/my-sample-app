pipeline {
    agent any

    environment {
        AWS_REGION     = "us-east-1"
        AWS_ACCOUNT_ID = "194526776035"
        ECR_REPO       = "mysample_appecr"
        IMAGE_TAG      = "${BUILD_NUMBER}"
        ECR_URL        = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

        CLUSTER_NAME   = "mysample-app-cluster"
        SERVICE_NAME   = "mysample-app-cluster-service"
        TASK_FAMILY    = "mysample-app-cluster"
        CONTAINER_NAME = "mysample-app"
    }

    stages {

        stage('Checkout') {
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

        stage('Register New Task Definition') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    bat """
                    aws ecs describe-task-definition --task-definition %TASK_FAMILY% --query taskDefinition > taskdef.json

                    powershell -Command "(Get-Content taskdef.json) ^
                    -replace '\"\\"image\\":.*?\"\",', '\\"image\\": \\"%ECR_URL%/%ECR_REPO%:%IMAGE_TAG%\\",' ^
                    | Set-Content taskdef-updated.json"

                    aws ecs register-task-definition --cli-input-json file://taskdef-updated.json
                    """
                }
            }
        }

        stage('Deploy to ECS Service') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    bat """
                    $revision=(aws ecs describe-task-definition --task-definition %TASK_FAMILY% --query "taskDefinition.revision" --output text)

                    aws ecs update-service ^
                        --cluster %CLUSTER_NAME% ^
                        --service %SERVICE_NAME% ^
                        --task-definition %TASK_FAMILY%:$revision
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment Successful!"
        }
        failure {
            echo "Deployment Failed!"
        }
    }
}
