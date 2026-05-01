pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "zakyaakram/jenkins-app:latest"
        KUBE_NAMESPACE = "ivolve"
    }

    stages {

        stage('Checkout') {
            steps {
                // مهم: ده بيضمن إن Jenkins يجيب repo الصح + branch الصح
                checkout scm
            }
        }

        stage('Debug Workspace') {
            steps {
                sh '''
                echo "===== WORKSPACE ====="
                pwd
                ls -la
                echo "===== SEARCH DEPLOYMENT FILE ====="
                find . -name deployment.yaml
                '''
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn -v || true'
                sh 'mvn clean test'
            }
        }

        stage('Build App') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker version
                docker build -t $DOCKER_IMAGE .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Delete Local Image') {
            steps {
                sh 'docker rmi $DOCKER_IMAGE || true'
            }
        }
        stage('Create Deployment File') {
    steps {
        sh '''
        cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins-app
  template:
    metadata:
      labels:
        app: jenkins-app
    spec:
      containers:
      - name: app
        image: $DOCKER_IMAGE
        ports:
        - containerPort: 8080
EOF
        '''
    }
}

   stage('Deploy to Kubernetes') {
    steps {
        withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
            sh '''
            export KUBECONFIG=$KUBECONFIG
            kubectl apply -f deployment.yaml -n $KUBE_NAMESPACE
            kubectl get pods -n $KUBE_NAMESPACE
            '''
        }
    }
}
    post {
        always {
            echo "Pipeline finished"
        }

        success {
            echo "Deployment SUCCESS 🚀"
        }

        failure {
            echo "Pipeline FAILED ❌"
        }
    }
}
