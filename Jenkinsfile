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



        stage('Check Tools') {

            steps {

                sh '''

                echo "Checking installed tools..."

                docker --version

                python --version || true

                node --version || true

                aws --version || true


                '''

            }
        }




        stage('Backend Test') {


            steps {


                sh '''

                cd backend


                python --version


                pip install --upgrade pip


                pip install -r requirements.txt


                python manage.py check


                python manage.py test


                '''


            }

        }





        stage('Frontend Build Test') {


            steps {


                sh '''

                cd frontend


                npm --version


                npm install


                npm run build


                '''


            }

        }





        stage('Build Docker Images') {


            steps {


                sh '''

                echo "Building backend image..."


                docker build \
                -t ${BACKEND_IMAGE}:${IMAGE_TAG} \
                ./backend



                echo "Building frontend image..."


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

                    echo "Logging into AWS ECR..."


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

                echo "Pushing images..."


                docker push ${BACKEND_IMAGE}:${IMAGE_TAG}


                docker push ${FRONTEND_IMAGE}:${IMAGE_TAG}



                echo "Creating latest tags..."


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



                    echo "Logging into ECR"



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

            Docker images pushed to ECR

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