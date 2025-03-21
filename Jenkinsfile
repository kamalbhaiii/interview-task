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
        stage('Setup Prometheus and Grafana') {
            steps {
                bat 'helm repo add prometheus-community https://prometheus-community.github.io/helm-charts'
                bat 'helm repo update'
                bat 'kubectl create namespace monitoring'
                bat 'helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring'
            }
        }
        stage('Remove Images locally') {
            steps {
                bat 'docker rmi backend:%IMAGE_TAG% || exit 0'
                bat 'docker rmi frontend:%IMAGE_TAG% || exit 0'
                bat 'docker rmi %BACKEND_IMAGE%:%IMAGE_TAG% || exit 0'
                bat 'docker rmi %FRONTEND_IMAGE%:%IMAGE_TAG% || exit 0'
            }
        }
        stage('Build image') {
            steps {
                bat 'docker build -t backend:%IMAGE_TAG% ./backend'
                bat 'docker build -t frontend:%IMAGE_TAG% ./frontend'
            }
        }
        stage('Docker login') {
            steps{
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        bat "echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin"
                    }
                }
            }
        }
        stage ('Push Backend Image') {
            steps {
                bat 'docker tag backend:%IMAGE_TAG% %BACKEND_IMAGE%:%IMAGE_TAG%'
                bat 'docker push %BACKEND_IMAGE%:%IMAGE_TAG%'
            }
        }
        stage ('Push Frontend Image') {
            steps {
                bat 'docker tag frontend:%IMAGE_TAG% %FRONTEND_IMAGE%:%IMAGE_TAG%'
                bat 'docker push %FRONTEND_IMAGE%:%IMAGE_TAG%'
            }
        }
        stage ('Remove images') {
            steps {
                bat 'docker rmi backend:%IMAGE_TAG% || exit 0'
                bat 'docker rmi frontend:%IMAGE_TAG% || exit 0'
                bat 'docker rmi %BACKEND_IMAGE%:%IMAGE_TAG% || exit 0'
                bat 'docker rmi %FRONTEND_IMAGE%:%IMAGE_TAG% || exit 0'
            }
        }
        stage('Run all k8s files file') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        bat 'kubectl apply -f k8s/'
                    }
                }
            }
        }
    }
}
