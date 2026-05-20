pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "605448157934.dkr.ecr.us-east-1.amazonaws.com/node-app"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/prashaakshi/node-app-devops.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t node-app .'
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh 'docker tag node-app:latest $ECR_REPO:$IMAGE_TAG'
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin 605448157934.dkr.ecr.us-east-1.amazonaws.com

                docker push $ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to App Host') {
            steps {
                sh '''
                ssh ec2-user@10.0.3.99 "
                docker pull $ECR_REPO:$IMAGE_TAG

                docker stop node-app || true
                docker rm node-app || true

                docker run -d --name node-app -p 80:3000 $ECR_REPO:$IMAGE_TAG
                "
                '''
            }
        }
    }
}