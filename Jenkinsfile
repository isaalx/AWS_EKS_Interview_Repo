pipeline {
    agent any

    tools {
        docker = 'docker-latest'
    }
    
    environment {
        BRANCH = "${params.BRANCH}"
        APP_NAME = "${params.APP_NAME}"
        ECR_REPO = "${params.ECR_REPO}"
        REPO_URL = "${params.REPO_URL}"
        AWS_REGION = "${params.AWS_REGION}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: $REPO_URL]])
            }
        }

        stage('Build') {
            steps {
                script {
                    sh 'docker build -f Dockerfile -t ${APP_NAME}:${BUILD_NUMBER} .'
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}'
                    sh 'docker push ${ECR_REPO}/${APP_NAME}:${BUILD_NUMBER}'
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    def helm_release_name = ""
                    sh 'helm upgrade --install ${APP_NAME} ./helmchart --set image.repository=${ECR_REPO}/${APP_NAME}:${BUILD_NUMBER}'
                }
            }
        }
    }
}
