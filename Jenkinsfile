pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        DOCKER_IMAGE = "zakyaakram52/jenkins-app:${BUILD_NUMBER}"
        KUBE_NAMESPACE = "ivolve"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/zakyaakram/jenkins-CI-CD.git'
            }
        }

        stage('Unit Test') {
            steps {
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
                sh 'docker build -t $DOCKER_IMAGE .'
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

        stage('Update Deployment File') {
            steps {
                sh '''
                sed -i "s|image: zakyaakram52/jenkins-app.*|image: $DOCKER_IMAGE|g" deployment.yaml
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f deployment.yaml -n $KUBE_NAMESPACE
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished'
        }

        success {
            echo 'Deployment SUCCESS 🚀'
        }

        failure {
            echo 'Pipeline FAILED ❌'
        }
    }
}
