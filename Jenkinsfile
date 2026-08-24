pipeline {

    agent any


    environment {

        AWS_REGION = "ap-south-1"

        AWS_ACCOUNT_ID = "YOUR_AWS_ACCOUNT_ID"


        BACKEND_IMAGE =
        "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/ecommerce-backend"


        FRONTEND_IMAGE =
        "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/ecommerce-frontend"


        IMAGE_TAG = "${BUILD_NUMBER}"


        AWS_CREDENTIALS = "aws-ecr-creds"


        EC2_KEY = "ec2-ssh-key"


        EC2_USER = "ubuntu"


        EC2_HOST = "YOUR_EC2_PUBLIC_IP"

    }



    options {

        skipDefaultCheckout(true)

        buildDiscarder(
            logRotator(
                numToKeepStr: '20'
            )
        )

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


                docker build \
                -t backend-test \
                ./backend
                
                echo "Running Django tests"

                docker run --rm backend-test \
                python manage.py check

                echo "backend test complete without database"

                
                

                


                '''

            }

        }




        stage('Frontend Build Test') {


            steps {


                sh '''

                echo "Building frontend test image"

                docker build \
                -t frontend-test \
                ./frontend
                
                echo "frontend build complete"

                "
                

                


                '''

            }

        }





        stage('Build Docker Images') {


            steps {


                sh '''

                echo "Building Backend Docker Image"


                docker build \
                -t ${BACKEND_IMAGE}:${IMAGE_TAG} \
                ./backend



                echo "Building Frontend Docker Image"


                docker build \
                -t ${FRONTEND_IMAGE}:${IMAGE_TAG} \
                ./frontend


                '''

            }

        }





        stage('Login AWS ECR') {


            steps {


                withAWS(
                    credentials: "${AWS_CREDENTIALS}",
                    region: "${AWS_REGION}"
                ) {


                    sh '''

                    echo "Logging into AWS ECR"


                    aws ecr get-login-password \
                    --region ${AWS_REGION} | \
                    docker login \
                    --username AWS \
                    --password-stdin \
                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com


                    '''

                }

            }

        }





        stage('Push Images To ECR') {


            steps {


                sh '''

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



                '''

            }

        }





        stage('Deploy To EC2') {


            steps {


                sshagent(
                    credentials: ["${EC2_KEY}"]
                ) {


                    sh """


                    echo "Connecting to EC2"



                    ssh -o StrictHostKeyChecking=no \
                    ${EC2_USER}@${EC2_HOST} << EOF



                    echo "Login into AWS ECR"



                    aws ecr get-login-password \
                    --region ${AWS_REGION} | \
                    docker login \
                    --username AWS \
                    --password-stdin \
                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com




                    echo "Pulling latest images"



                    docker pull \
                    ${BACKEND_IMAGE}:latest



                    docker pull \
                    ${FRONTEND_IMAGE}:latest




                    cd ecommerce-project




                    docker compose down




                    docker compose up -d




                    echo "Deployment completed"



                    exit


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

            PIPELINE SUCCESSFUL

            Images pushed to AWS ECR

            Application deployed on EC2

            =============================

            """

        }




        failure {


            echo """

            =============================

            PIPELINE FAILED

            Check Jenkins console logs

            =============================

            """

        }



        always {


            echo """

            Build Number:
            ${BUILD_NUMBER}

            Status:
            ${currentBuild.currentResult}

            """

        }


    }


}