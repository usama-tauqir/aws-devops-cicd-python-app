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

            agent {

                docker {

                    image 'python:3.9-slim'

                    reuseNode true

                }
            }


            steps {

                dir('backend') {


                    sh '''

                    pip install --upgrade pip

                    pip install -r requirements.txt


                    python manage.py check


                    python manage.py test


                    '''

                }
            }
        }




        stage('Frontend Build Test') {


            agent {

                docker {

                    image 'node:16-alpine'

                    reuseNode true

                }
            }


            steps {

                dir('frontend') {


                    sh '''

                    npm install

                    npm run build

                    '''

                }
            }
        }




        stage('Build Docker Images') {


            steps {


                sh '''

                docker build \
                -t ${BACKEND_IMAGE}:${IMAGE_TAG} \
                ./backend



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

                docker push ${BACKEND_IMAGE}:${IMAGE_TAG}

                docker push ${FRONTEND_IMAGE}:${IMAGE_TAG}



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


                    ssh -o StrictHostKeyChecking=no \
                    ${EC2_USER}@${EC2_HOST} << EOF


                    docker login \
                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com



                    docker pull \
                    ${BACKEND_IMAGE}:latest



                    docker pull \
                    ${FRONTEND_IMAGE}:latest



                    cd ecommerce-project



                    docker compose down



                    docker compose up -d



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

            =========================

            PIPELINE SUCCESSFUL

            Images pushed to AWS ECR

            Application deployed

            =========================

            """

        }



        failure {


            echo """

            =========================

            PIPELINE FAILED

            Check Jenkins logs

            =========================

            """

        }


    }

}