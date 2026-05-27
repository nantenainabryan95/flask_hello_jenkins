pipeline {
    agent any

    triggers {
        pollSCM('* * * * *')
    }

    stages {
        stage('Test') {
            steps {
                echo 'Test stage'
                sh "pip3 install -r requirements.txt || true"
                sh "python3 test.py || true"
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'
                sh "docker build -t pythontest:latest . || true"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes'
                sh "kubectl apply -f ./kubernetes/deployment.yaml"
                sh "kubectl apply -f ./kubernetes/service.yaml"
            }
        }
    }

    post {
        success {
            echo 'Pipeline reussi !'
        }
        failure {
            echo 'Pipeline echoue !'
        }
    }
}