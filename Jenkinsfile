pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'balajirocks/test-react-app'
        IMAGE_TAG = 'latest'
        KUBECONFIG = '/var/run/secrets/kubernetes.io/serviceaccount/kubeconfig'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git credentialsId: 'github_token', url: 'https://github.com/BalajiRocks/react-app.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .'
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', 'docker-hub-credentials-id') {
                        sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
                    }
                }
            }
        }
        
	   stage('Cleanup') {
            steps {
                script {
                    sh "docker rmi ${DOCKER_IMAGE}:${IMAGE_TAG}"
                }
            }
        }

        stage('Create Deployment YAML') {
            steps {
                script {
                    sh """
                    cat <<EOF > deployment.yaml
                    apiVersion: apps/v1
                    kind: Deployment
                    metadata:
                      name: react-app-deployment
                    spec:
                      replicas: 1
                      selector:
                        matchLabels:
                          app: react-app
                      template:
                        metadata:
                          labels:
                            app: react-app
                        spec:
                          containers:
                          - name: react-app
                            image: ${DOCKER_IMAGE}:${IMAGE_TAG}
                            ports:
                            - containerPort: 5000
                    EOF
                    """
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    sh 'kubectl get pods -o wide'
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

