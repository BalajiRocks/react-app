pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'kunalmac25/react-example-image'
        IMAGE_TAG = 'latest'
        KUBECONFIG = '/home/jenkins/.kube/config'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git credentialsId: 'github_token', url: 'https://github.com/BalajiRocks/react-app.git', branch: 'main'
            }
        }

        stage('Install kubectl') {
            steps {
                script {
                    sh '''
                    mkdir -p /home/jenkins/bin
                    curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.31.0/bin/linux/amd64/kubectl
                    chmod +x kubectl
                    mv kubectl /home/jenkins/bin/kubectl
                    '''
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    sh '/home/jenkins/bin/kubectl apply -f deployment.yaml'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    sh '/home/jenkins/bin/kubectl get pods -o wide'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
