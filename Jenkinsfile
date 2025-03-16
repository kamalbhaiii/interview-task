pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub'
        FRONTEND_IMAGE = 'kamalbhaiii/frontend'
        BACKEND_IMAGE = 'kamalbhaiii/backend'
        IMAGE_TAG = 'latest'
    }

    stages{
        stage('Checkout Code') {
            steps{
                git 'https://github.com/kamalbhaiii/interview-task.git'
            }
        }
        stage('Remove Images locally') {
            steps {
                bat 'docker rmi backend:${IMAGE_TAG} || TRUE'
                bat 'docker rmi frontend:${IMAGE_TAG} || TRUE'
                bat 'docker rmi ${BACKEND_IMAGE}:${IMAGE_TAG} || TRUE'
                bat 'docker rmi ${FRONTEND_IMAGE}:${IMAGE_TAG} || TRUE'
            }
        }
        stage('Build image') {
            steps {
                bat 'docker build -t ${BACKEND_IMAGE}:${IMAGE_TAG} /backend'
                bat 'docker build -t ${FRONTEND_IMAGE}:${IMAGE_TAG} /frontend'
            }
        }
        stage('Docker login') {
            steps{
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    }
                }
            }
        }
        stage ('Push Backend Image') {
            steps {
                bat 'docker tag backend:${IMAGE_TAG} ${BACKEND_IMAGE}:${IMAGE_TAG}'
                bat 'docker push ${BACKEND_IMAGE}:${IMAGE_TAG}'
            }
        }
        stage ('Push Frontend Image') {
            steps {
                bat 'docker tag frontend:${IMAGE_TAG} ${FRONTEND_IMAGE}:${IMAGE_TAG}'
                bat 'docker push ${FRONTEND_IMAGE}:${IMAGE_TAG}'
            }
        }
        stage ('Remove images') {
            steps {
                bat 'docker rmi backend:${IMAGE_TAG} || TRUE'
                bat 'docker rmi frontend:${IMAGE_TAG} || TRUE'
                bat 'docker rmi ${BACKEND_IMAGE}:${IMAGE_TAG} || TRUE'
                bat 'docker rmi ${FRONTEND_IMAGE}:${IMAGE_TAG} || TRUE'
            }
        }
        stage('Run compose file') {
            steps {
                bat 'docker-compose -f docker-compose.yaml up -d'
            }
        }
    }
}