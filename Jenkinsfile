pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        AWS_ACCOUNT_ID = "096942125249"
        AWS_CREDENTIALS = "aws-ecr-creds"
        BACKEND_IMAGE = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/ecommerce-backend"
        FRONTEND_IMAGE = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/ecommerce-frontend"
        IMAGE_TAG = "${BUILD_NUMBER}"
        EC2_KEY = "ec2-ssh-key"
        EC2_USER = "ubuntu"
        EC2_HOST = "43.204.214.232"
    }

    options {
        skipDefaultCheckout(true)
        buildDiscarder(logRotator(numToKeepStr: '20'))
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Backend Test') {
            steps {
                sh '''
                    echo "Building backend test image"
                    docker build -t backend-test ./backend
                    echo "Running Django tests"
                    docker run --rm backend-test python manage.py check
                    echo "backend test complete without database"
                '''
            }
        }

        stage('Frontend Build Test') {
            steps {
                sh '''
                    echo "Building frontend test image"
                    docker build -t frontend-test ./frontend
                    echo "frontend build complete"
                '''
            }
        }

        stage('Create ECR Repositories') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS}"]]) {
                    sh """
                        echo "Creating ECR repositories if they don't exist"
                        
                        if aws ecr describe-repositories --repository-names ecommerce-backend --region ${AWS_REGION} 2>&1 | grep -q "RepositoryNotFoundException"; then
                            echo "Creating ecommerce-backend repository"
                            aws ecr create-repository --repository-name ecommerce-backend --region ${AWS_REGION}
                        else
                            echo "ecommerce-backend repository already exists"
                        fi
                        
                        if aws ecr describe-repositories --repository-names ecommerce-frontend --region ${AWS_REGION} 2>&1 | grep -q "RepositoryNotFoundException"; then
                            echo "Creating ecommerce-frontend repository"
                            aws ecr create-repository --repository-name ecommerce-frontend --region ${AWS_REGION}
                        else
                            echo "ecommerce-frontend repository already exists"
                        fi
                    """
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh """
                    echo "Building Backend Docker Image"
                    docker build -t ${BACKEND_IMAGE}:${IMAGE_TAG} ./backend

                    echo "Building Frontend Docker Image"
                    docker build -t ${FRONTEND_IMAGE}:${IMAGE_TAG} ./frontend
                """
            }
        }

        stage('Login AWS ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS}"]]) {
                    sh """
                        echo "Logging into AWS ECR"
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    """
                }
            }
        }

        stage('Push Images To ECR') {
            steps {
                sh """
                    echo "Pushing backend image"
                    docker push ${BACKEND_IMAGE}:${IMAGE_TAG}

                    echo "Pushing frontend image"
                    docker push ${FRONTEND_IMAGE}:${IMAGE_TAG}

                    echo "Creating latest tags"
                    docker tag ${BACKEND_IMAGE}:${IMAGE_TAG} ${BACKEND_IMAGE}:latest
                    docker tag ${FRONTEND_IMAGE}:${IMAGE_TAG} ${FRONTEND_IMAGE}:latest

                    docker push ${BACKEND_IMAGE}:latest
                    docker push ${FRONTEND_IMAGE}:latest
                """
            }
        }

        stage('Deploy To EC2') {
            steps {
                sshagent(credentials: ["${EC2_KEY}"]) {
                    sh """
                        echo "Connecting to EC2"
                        
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << 'EOF'
                            echo "Login into AWS ECR"
                            aws ecr get-login-password --region ${AWS_REGION} | \
                            docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

                            echo "Pulling latest images"
                            docker pull ${BACKEND_IMAGE}:latest
                            docker pull ${FRONTEND_IMAGE}:latest

                            echo "Stopping old containers"
                            docker stop \$(docker ps -q) 2>/dev/null || true
                            docker rm \$(docker ps -aq) 2>/dev/null || true

                            echo "Running backend container"
                            docker run -d --name backend -p 8000:8000 ${BACKEND_IMAGE}:latest

                            echo "Running frontend container"
                            docker run -d --name frontend -p 80:80 ${FRONTEND_IMAGE}:latest

                            echo "Deployment completed"
EOF
                    """
                }
            }
        }
    }

    post {
        success {
            echo """
                =============================
                PIPELINE SUCCESSFUL ✅
                Images pushed to AWS ECR
                Application deployed on EC2
                =============================
            """
        }

        failure {
            echo """
                =============================
                PIPELINE FAILED ❌
                Check Jenkins console logs
                =============================
            """
        }

        always {
            echo """
                Build Number: ${BUILD_NUMBER}
                Status: ${currentBuild.currentResult}
            """
        }
    }
}