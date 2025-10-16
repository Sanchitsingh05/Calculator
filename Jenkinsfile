pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "sanchit0305/calculator-app"
        DOCKER_CREDENTIALS_ID = "dockerhub-creds"
        HELM_RELEASE_NAME = "calculator-release"
        HELM_CHART_NAME = "to-do-chart"
       // KUBE_CONTEXT = "/home/ubuntu/.kube/config"
    }

    triggers {
        githubPush() // Triggered when PR is merged to main
    }

    options {
        skipDefaultCheckout()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_CREDENTIALS_ID) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to Kubernetes with Helm') {
            steps {
                script {
                    sh """
                        helm upgrade --install ${HELM_RELEASE_NAME} ./${HELM_CHART_NAME} \
                        --set image.repository=${DOCKER_IMAGE} \
                        --set image.tag=${env.BUILD_NUMBER} \
                        --kubeconfig /home/ubuntu/.kube/config
                    """
                }
            }
        }
        stage('Deploy to Kubernetes (Blue)') {
            steps {
                script {
                    sh """
                       helm upgrade --install calculator-release ./to-do-chart \
                       --set image.repository=${DOCKER_IMAGE} \
                       --set image.tag=${env.BUILD_NUMBER} \
                       --kube-context minikube
                    """
                   }
              }
        } 
    }
}
