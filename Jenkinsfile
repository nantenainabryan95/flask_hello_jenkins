pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: python
    image: python:3.9
    command: ["cat"]
    tty: true
  - name: docker
    image: docker:24.0.9
    command: ["cat"]
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
  - name: kubectl
    image: lachlanevenson/k8s-kubectl:v1.17.2
    command:
    - cat
    tty: true
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
'''
        }
    }

    stages {
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