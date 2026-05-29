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
    imagePullPolicy: IfNotPresent
    command: ['/bin/cat']
    tty: true
    securityContext:
      privileged: true
    env:
    - name: DOCKER_TLS_CERTDIR
      value: ""
  - name: kubectl
    image: bitnami/kubectl:latest
    imagePullPolicy: IfNotPresent
    command: ['/bin/cat']
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
        stage('Test') {
            steps {
                container('python') {
                    sh '''
                        pip install --upgrade pip
                        pip install -r requirements.txt
                        python test.py
                    '''
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    sh '''
                        until docker info; do sleep 2; done
                        docker build -t pythontest:latest .
                        docker tag pythontest:latest localhost:4000/pythontest:latest
                        docker push localhost:4000/pythontest:latest
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