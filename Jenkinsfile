pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = 'ignatiusnobel13/aws-devops-demo'
        MINIKUBE_IP = '13.203.205.208'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'develop', url: 'https://github.com/Ignatius-Nobel/aws-springboot-devops.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} ."
                sh "docker tag ${DOCKERHUB_REPO}:${IMAGE_TAG} ${DOCKERHUB_REPO}:latest"
            }
        }

        stage('Docker Hub Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh '''
                        echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}
                        docker push ${DOCKERHUB_REPO}:latest
                    '''
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sshagent(['minikube-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${MINIKUBE_IP} '
                            export KUBECONFIG=/home/ubuntu/.kube/config
                            kubectl set image deployment/aws-devops-demo aws-devops-demo=${DOCKERHUB_REPO}:${IMAGE_TAG} --record
                            kubectl rollout status deployment/aws-devops-demo
                        '
                    """
                }
            }
        }
    }
}
