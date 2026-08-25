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
                        echo "Checking ECR repositories"
                        
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

                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ec2-ssh-key',
                        keyFileVariable: 'SSH_KEY'
                    )
                ]) {

                    sh '''
                    ssh -o StrictHostKeyChecking=no -i $SSH_KEY ubuntu@43.204.214.232 'bash -s' <<'REMOTE_SCRIPT'

                    set -e

                    echo "=== Starting Deployment ==="

                    echo "Login into AWS ECR"

                    aws ecr get-login-password \
                    --region ap-south-1 | \
                    docker login \
                    --username AWS \
                    --password-stdin \
                    096942125249.dkr.ecr.ap-south-1.amazonaws.com


                    echo "Stopping old containers"

                    docker rm -f ecommerce-backend || true
                    docker rm -f ecommerce-frontend || true


                    echo "Cleaning unused Docker resources"

                    docker system prune -af


                    echo "Pulling latest images"

                    docker pull 096942125249.dkr.ecr.ap-south-1.amazonaws.com/ecommerce-backend:latest

                    docker pull 096942125249.dkr.ecr.ap-south-1.amazonaws.com/ecommerce-frontend:latest


                    echo "Starting backend"

                    docker run -d \
                    --name ecommerce-backend \
                    -p 8000:8000 \
                    096942125249.dkr.ecr.ap-south-1.amazonaws.com/ecommerce-backend:latest


                    echo "Starting frontend"

                    docker run -d \
                    --name ecommerce-frontend \
                    -p 80:80 \
                    096942125249.dkr.ecr.ap-south-1.amazonaws.com/ecommerce-frontend:latest


                    echo "Checking containers"

                    docker ps


                    echo "=== Deployment Completed Successfully ==="

                    REMOTE_SCRIPT
                    '''

                }

            }

        }
    }

    post {
        success {
            echo """
                =============================
                🎉 PIPELINE SUCCESSFUL ✅
                📦 Images pushed to AWS ECR
                🚀 Application deployed on EC2
                🌐 Backend: http://${EC2_HOST}:8000
                🌐 Frontend: http://${EC2_HOST}
                =============================
            """
        }

        failure {
            echo """
                =============================
                ❌ PIPELINE FAILED
                Please check Jenkins console logs
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