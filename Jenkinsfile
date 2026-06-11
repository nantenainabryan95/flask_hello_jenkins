pipeline {
    agent any

    triggers {
        pollSCM('* * * * *')
    }

    stages {
        stage('Test python') {
            steps {
                sh "pip install -r requirements.txt"
                sh "python test.py"
            }
        }

        stage('Build image') {
            steps {
                sh "docker build -t localhost:4000/pythontest:latest ."
                sh "docker push localhost:4000/pythontest:latest"
            }
        }

        stage('Deploy') {
            steps {
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
