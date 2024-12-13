def EC2_PUBLIC_IP = ""
def RDS_ENDPOINT = ""
def DEPLOYER_KEY_URI = ""
def IMAGE_REPO_FRONTEND = "205930615645.dkr.ecr.us-east-1.amazonaws.com/enis-app/frontend-app-1.0"
def IMAGE_REPO_BACKEND = "205930615645.dkr.ecr.us-east-1.amazonaws.com/enis-app/backend-app-1.0"

pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
        ECR_REPO_URL = '205930615645.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPO_NAME = 'enis-app'
        IMAGE_REPO = "${ECR_REPO_URL}/${ECR_REPO_NAME}"
        AWS_REGION = "us-east-1"
    }
    stages {
        stage('Provision Server and Database') {
            steps {
                script {
                    dir('my-terraform-project/remote_backend') {
                        sh "terraform init"
                        sh "terraform apply --auto-approve"
                    }
                    dir('my-terraform-project') {
                        sh "terraform init"
                        sh "terraform plan -lock=false"
                        sh "terraform apply -lock=false --auto-approve"

                        EC2_PUBLIC_IP = sh(
                            script: '''terraform output instance_details | grep "instance_public_ip" | awk '{print $3}' | tr -d '"' ''',
                            returnStdout: true
                        ).trim()

                        RDS_ENDPOINT = sh(
                            script: '''
                                terraform output rds_endpoint | grep "endpoint" | awk -F'=' '{print $2}' | tr -d '[:space:]"' | sed 's/:3306//'
                            ''',
                            returnStdout: true
                        ).trim()

                        DEPLOYER_KEY_URI = sh(
                            script: '''terraform output deployer_key_s3_uri | tr -d '"' ''',
                            returnStdout: true
                        ).trim()

                        echo "EC2 Public IP: ${EC2_PUBLIC_IP}"
                        echo "RDS Endpoint: ${RDS_ENDPOINT}"
                        echo "Deployer Key URI: ${DEPLOYER_KEY_URI}"
                    }
                }
            }
        }
        stage('Update Frontend Configuration') {
            steps {
                script {
                    dir('enis-app-tp/frontend/src') {
                        writeFile file: 'config.js', text: """
                            export const API_BASE_URL = 'http://${EC2_PUBLIC_IP}:8000';
                        """
                        sh '''
                            echo "Contents of config.js after update:"
                            cat config.js
                        '''
                    }
                }
            }
        }
        stage('Update Backend Configuration') {
            steps {
                script {
                    dir('enis-app-tp/backend/backend') {
                        sh '''
                            if [ -f "settings.py" ]; then
                                echo "Found settings.py at $(pwd)"
                            else
                                echo "settings.py not found in $(pwd)!"
                                exit 1
                            fi
                        '''
                        sh """
                            sed -i "/'HOST':/c\\            'HOST': '${RDS_ENDPOINT}'," settings.py
                        """
                        sh '''
                            echo "DATABASES section of settings.py after update:"
                            sed -n '/DATABASES = {/,/^}/p' settings.py
                        '''
                    }
                }
            }
        }
        stage('Create Database in RDS') {
            steps {
                script {
                    sh """
                    mysql -h ${RDS_ENDPOINT} -P 3306 -u dbuser -pDBpassword2024 -e "CREATE DATABASE IF NOT EXISTS enis_tp;"
                    mysql -h ${RDS_ENDPOINT} -P 3306 -u dbuser -pDBpassword2024 -e "SHOW DATABASES;"
                    """
                }
            }
        }
        stage('Build Frontend Docker Image') {
            steps {
                script {
                    echo 'Building Frontend Docker Image...'
                    def frontendImage = docker.build('frontend-app', 'enis-app-tp/frontend')
                    echo "Built Image: ${frontendImage.id}"
                }
            }
        }
        stage('Build Backend Docker Image') {
            steps {
                script {
                    echo 'Building Backend Docker Image...'
                    def backendImage = docker.build('backend-app', 'enis-app-tp/backend')
                    echo "Built Image: ${backendImage.id}"
                }
            }
        }
        stage('Login to AWS ECR') {
            steps {
                script {
                    sh """
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO_URL}
                    """
                }
            }
        }
        stage('Tag and Push Frontend Image') {
            steps {
                script {
                    echo 'Tagging and pushing Frontend Image...'
                    sh "docker tag frontend-app:latest $IMAGE_REPO_FRONTEND"
                    sh "docker push $IMAGE_REPO_FRONTEND"
                }
            }
        }
        stage('Tag and Push Backend Image') {
            steps {
                script {
                    echo 'Tagging and pushing Backend Image...'
                    sh "docker tag backend-app:latest $IMAGE_REPO_BACKEND"
                    sh "docker push $IMAGE_REPO_BACKEND"
                }
            }
        }
    }
}
