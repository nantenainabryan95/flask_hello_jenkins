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
    imagePullPolicy: IfNotPresent
    command: ["cat"]
    tty: true
  - name: docker
    image: docker:24.0.9-dind
    command: ["dockerd", "-H", "tcp://127.0.0.1:2375", "--tls=false"]
    tty: true
    securityContext:
      privileged: true
    env:
    - name: DOCKER_HOST
      value: tcp://localhost:2375
  - name: kubectl
    image: bitnami/kubectl:latest
    imagePullPolicy: IfNotPresent
    command: ["cat"]
    tty: true
  # Plus besoin du volume docker-sock avec DinD !
'''
        }
    }
    
    triggers {
        pollSCM('* * * * *')
    }
    
    stages {
        stage('Debug - List files') {
            steps {
                container('python') {
                    sh '''
                        echo "=== Current directory ==="
                        pwd
                        echo "=== Files in workspace ==="
                        ls -la
                    '''
                }
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
                        until docker info > /dev/null 2>&1; do
                            echo "Waiting for Docker daemon to start..."
                            sleep 2
                        done
                        echo "Docker is ready!"
                        docker build -t localhost:5001/pythontest:latest .
                        docker push localhost:5001/pythontest:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh '''
                        echo "Deploying to Kubernetes..."
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