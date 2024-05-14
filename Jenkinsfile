pipeline {
    agent any
    environment {
        DOCKERHUB_USERNAME = 'huytumoneyforward'
        APP_NAME = 'gitops-demo-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + '/' + "${APP_NAME}"
        REGISTRY_CREDS = 'dockerhub'
    }
    stages {
        stage('Cleanup Workspace') {
            steps {
                script {
                    cleanWs()
                }
            }
        }
        stage('Checkout SCM') {
            steps {
                git credentialsId: 'github',
                url: 'https://github.com/huytuchim789/gitops.git',
                branch: 'dev'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    withCredentials([usernameColonPassword(credentialsId: 'dockerhub', variable: 'user')]) {
                        docker_image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', "${REGISTRY_CREDS}") {
                        docker_image.push("${BUILD_NUMBER}")
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage('Delete Docker Image') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
        stage('Updating Kubernetes deployment file') {
            steps {
                sh 'cat deployment.yml'
                sh "sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yml"
                sh 'cat deployment.yml'
            }
        }
        stage('Push the changed deployment file to Git') {
            steps {
                script {
                    sh """
                    git config --global user.name "huytuchim789"
                    git config --global user.email "tranhuytu242000@gmail.com"
                    git add deployment.yml
                    git commit -m 'Updated the deployment file' """
                    withCredentials([usernamePassword(credentialsId: 'github', passwordVariable: 'pass', usernameVariable: 'user')]) {
                        sh "git push https://$user:$pass@github.com/huytuchim/gitops.git dev"
                    }
                }
            }
        }
    }
}
