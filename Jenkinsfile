pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '769566xxxxxx'
        ECR_REPO_F = 'meanstack/frontend'
        ECR_REPO_B = 'meanstack/backend'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }

    stages {

        stage('Checkout') {
            steps {
                git url: "https://github.com/kapilgole1/Mean-stack.git", branch: "main"
                echo "Git cloning successful"
            }
        }

        stage('Build Docker Images') {
            steps {
                sh "docker build -t ${ECR_REGISTRY}/${ECR_REPO_F}:${IMAGE_TAG} ./frontend"
                sh "docker build -t ${ECR_REGISTRY}/${ECR_REPO_B}:${IMAGE_TAG} ./backend"
                echo "Build complete"
            }
        }

        stage('AWS Login to ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'AWS-CRED',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh '''
                        echo "Logging in to ECR..."
                        aws ecr get-login-password --region $AWS_REGION | \
                        docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                        
                        echo "Verifying ECR access:"
                        aws ecr describe-repositories --region $AWS_REGION
                    '''
                }
            }
        }

        stage('Push Docker Images to ECR') {
            steps {
                sh """
                    docker push ${ECR_REGISTRY}/${ECR_REPO_F}:${IMAGE_TAG}
                    docker push ${ECR_REGISTRY}/${ECR_REPO_B}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }

        failure {
            echo '❌ Pipeline failed. Please check logs above.'
        }

        always {
            echo '🧹 Cleaning up Docker images...'
            sh '''
                docker image prune -f
            '''
        }
    }
}
