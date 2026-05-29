cd ~/tp_jenkins/flask_app
cat > Jenkinsfile << 'EOF'
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
    command:
    - cat
    tty: true
  - name: docker
    image: docker:24.0.9
    imagePullPolicy: IfNotPresent
    command:
    - cat
    tty: true
    env:
    - name: DOCKER_HOST
      value: tcp://host.docker.internal:2375
  - name: kubectl
    image: bitnami/kubectl:latest
    imagePullPolicy: IfNotPresent
    command:
    - cat
    tty: true
'''
        }
    }

    triggers {
        pollSCM('* * * * *')
    }

    stages {

        stage('1 - Debug workspace') {
            steps {
                container('python') {
                    sh '''
                        echo "=== Repertoire courant ==="
                        pwd
                        ls -la
                    '''
                }
            }
        }

        stage('2 - Test Python') {
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

        stage('3 - Build et Push image Docker') {
            steps {
                container('docker') {
                    sh '''
                        echo "=== Test connexion Docker ==="
                        docker version
                        echo "=== Build image ==="
                        docker build -t host.docker.internal:4000/pythontest:latest .
                        echo "=== Push image ==="
                        docker push host.docker.internal:4000/pythontest:latest
                    '''
                }
            }
        }

        stage('4 - Deploy sur Kubernetes') {
            steps {
                container('kubectl') {
                    sh '''
                        kubectl apply -f ./kubernetes/deployment.yaml
                        kubectl apply -f ./kubernetes/service.yaml
                        kubectl rollout status deployment/pythontest --timeout=90s
                    '''
                }
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
EOF