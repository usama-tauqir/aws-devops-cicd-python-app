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


                    echo "Running Django system check"

                    docker run --rm backend-test python manage.py check


                    echo "Backend test completed"

                '''

            }

        }




        stage('Frontend Build Test') {

            steps {

                sh '''

                    echo "Building frontend test image"


                    docker build -t frontend-test ./frontend


                    echo "Frontend build completed"

                '''

            }

        }





        stage('Create ECR Repositories') {


            steps {


                withCredentials([

                    [$class: 'AmazonWebServicesCredentialsBinding',

                    credentialsId: "${AWS_CREDENTIALS}"]

                ]) {


                    sh """

                    echo "Checking ECR repositories"



                    aws ecr describe-repositories \

                    --repository-names ecommerce-backend \

                    --region ${AWS_REGION} \

                    || aws ecr create-repository \

                    --repository-name ecommerce-backend \

                    --region ${AWS_REGION}




                    aws ecr describe-repositories \

                    --repository-names ecommerce-frontend \

                    --region ${AWS_REGION} \

                    || aws ecr create-repository \

                    --repository-name ecommerce-frontend \

                    --region ${AWS_REGION}


                    """

                }

            }

        }





        stage('Build Docker Images') {


            steps {


                sh """


                echo "Building backend image"


                docker build \

                -t ${BACKEND_IMAGE}:${IMAGE_TAG} \

                ./backend




                echo "Building frontend image"


                docker build \

                -t ${FRONTEND_IMAGE}:${IMAGE_TAG} \

                ./frontend



                """

            }

        }






        stage('Login AWS ECR') {


            steps {


                withCredentials([

                    [$class: 'AmazonWebServicesCredentialsBinding',

                    credentialsId: "${AWS_CREDENTIALS}"]

                ]) {



                    sh """


                    echo "Login into AWS ECR"



                    aws ecr get-login-password \

                    --region ${AWS_REGION} | \

                    docker login \

                    --username AWS \

                    --password-stdin \

                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com



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



                docker tag \

                ${BACKEND_IMAGE}:${IMAGE_TAG} \

                ${BACKEND_IMAGE}:latest




                docker tag \

                ${FRONTEND_IMAGE}:${IMAGE_TAG} \

                ${FRONTEND_IMAGE}:latest





                docker push ${BACKEND_IMAGE}:latest


                docker push ${FRONTEND_IMAGE}:latest



                """

            }

        }





        stage('Deploy To EC2') {


            steps {



                withCredentials([

                    sshUserPrivateKey(

                        credentialsId: "${EC2_KEY}",

                        keyFileVariable: "SSH_KEY"

                    )

                ]) {



                    sh """


                    ssh \

                    -o StrictHostKeyChecking=no \

                    -i \$SSH_KEY \

                    ${EC2_USER}@${EC2_HOST} << EOF



                    set -e



                    echo "===== EC2 Deployment Started ====="




                    echo "Login into ECR"



                    aws ecr get-login-password \

                    --region ${AWS_REGION} | \

                    docker login \

                    --username AWS \

                    --password-stdin \

                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com





                    echo "Pulling latest images"



                    docker pull ${BACKEND_IMAGE}:latest


                    docker pull ${FRONTEND_IMAGE}:latest





                    echo "Stopping old containers"



                    docker stop ecommerce-backend || true

                    docker rm ecommerce-backend || true




                    docker stop ecommerce-frontend || true

                    docker rm ecommerce-frontend || true





                    echo "Starting backend container"



                    docker run -d \

                    --name ecommerce-backend \

                    -p 8000:8000 \

                    ${BACKEND_IMAGE}:latest





                    echo "Starting frontend container"



                    docker run -d \

                    --name ecommerce-frontend \

                    -p 80:80 \

                    ${FRONTEND_IMAGE}:latest






                    echo "Cleaning unused images"



                    docker image prune -f





                    echo "Running Containers"



                    docker ps





                    echo "===== Deployment Completed ====="




                    EOF



                    """

                }

            }

        }


    }




    post {


        success {


            echo """

            ==============================

            PIPELINE SUCCESSFUL

            Images pushed to AWS ECR

            Application deployed on EC2


            Backend:

            http://${EC2_HOST}:8000


            Frontend:

            http://${EC2_HOST}


            ==============================

            """

        }





        failure {


            echo """

            ==============================

            PIPELINE FAILED

            Check Jenkins logs

            ==============================

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