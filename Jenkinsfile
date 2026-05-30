pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: python
    image: python:3.9
    imagePullPolicy: IfNotPresent
    command: ['/bin/cat']
    tty: true
  - name: docker
    image: docker:24.0.9-dind
    command: ["dockerd", "-H", "tcp://127.0.0.1:2375", "--tls=false", "--insecure-registry=host.docker.internal:5001"]
    tty: true
    securityContext:
      privileged: true
    env:
    - name: DOCKER_HOST
      value: tcp://localhost:2375
  - name: kubectl
    image: bitnami/kubectl:latest
    imagePullPolicy: IfNotPresent
    command:
    - sleep
    - "999999"
    tty: true
    securityContext:
      runAsUser: 0
"""
        }
    }
    triggers {
        pollSCM('* * * * *')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Test Python') {
            steps {
                container('python') {
                    sh '''
                        pip install --upgrade pip
                        pip install -r requirements.txt
                        python test.py -v
                    '''
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    sh '''
                        echo "Waiting for Docker daemon..."
                        sleep 10
                        docker build -t host.docker.internal:5001/pythontest:latest .
                        docker push host.docker.internal:5001/pythontest:latest
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh '''
                        kubectl apply -f ./kubernetes/deployment.yaml
                        kubectl apply -f ./kubernetes/service.yaml
                        kubectl rollout status deployment/pythontest
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline réussi !'
        }
        failure {
            echo 'Pipeline échoué !'
        }
    }
}