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
    image: docker:24.0.9
    imagePullPolicy: IfNotPresent
    command: ["cat"]
    tty: true
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-sock
  - name: kubectl
    image: bitnami/kubectl:latest
    imagePullPolicy: IfNotPresent
    command: ["cat"]
    tty: true
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
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
                        echo "=== Check test.py exists ==="
                        test -f test.py && echo "test.py found!" || echo "test.py NOT found!"
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
                        echo "=== Verifying Docker socket ==="
                        ls -la /var/run/docker.sock
                        echo "=== Docker version ==="
                        docker version
                        echo "=== Building image ==="
                        docker build -t localhost:4000/pythontest:latest .
                        echo "=== Pushing image ==="
                        docker push localhost:4000/pythontest:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh '''
                        echo "=== Applying manifests ==="
                        kubectl apply -f ./kubernetes/deployment.yaml
                        kubectl apply -f ./kubernetes/service.yaml
                        echo "=== Waiting for rollout ==="
                        kubectl rollout status deployment/pythontest --timeout=60s
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