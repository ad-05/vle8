pipeline {
    agent any

    environment {
        IMAGE = "myapp:v2"
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t myapp:v2 .'
            }
        }

        stage('Load Image into Minikube') {
            steps {
                sh 'minikube image load myapp:v2'
            }
        }

        stage('Deploy Green') {
            steps {
                sh 'kubectl apply -f deployment-green.yaml'
                sh 'kubectl rollout status deployment/myapp-green --timeout=120s'
            }
        }

        stage('Test Green') {
            steps {
                sh '''
                kubectl exec deployment/myapp-green -- \
                  wget -qO- http://127.0.0.1 | grep -q "Version 2"
                '''
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                sh '''
                kubectl patch service myapp-service \
                  -p '{"spec":{"selector":{"app":"myapp","color":"green"}}}'
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh 'kubectl get deployments'
                sh 'kubectl get pods'
                sh 'kubectl describe service myapp-service'
                sh 'kubectl get endpoints myapp-service'
            }
        }
    }
}
