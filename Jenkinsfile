pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "bhau2707"   // replace with your DockerHub username
        IMAGE_NAME = "${DOCKERHUB_USER}/webapp-demo"
        TAG = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
        KUBE_DIR = "k8s"
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build Docker Image') {
            steps { sh "docker build -t ${IMAGE_NAME}:${TAG} ." }
        }

        stage('Push DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push ${IMAGE_NAME}:${TAG}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                sed -i "s|image:.*|image: ${IMAGE_NAME}:${TAG}|" ${KUBE_DIR}/deployment.yaml
                kubectl apply -f ${KUBE_DIR}/deployment.yaml
                kubectl apply -f ${KUBE_DIR}/service.yaml
                """
            }
        }
    }
}
