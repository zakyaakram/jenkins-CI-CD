pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "zakyaakram52/jenkins-app"
        KUBE_NAMESPACE = "ivolve"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/zakyaakram/jenkins-CI-CD.git''
            }
        }

        stage('Run Unit Test') {
            steps {
                sh 'mvn clean test -U'
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push to Docker Hub') {
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

        stage('Remove Local Image') {
            steps {
                sh 'docker rmi $DOCKER_IMAGE || true'
            }
        }

        stage('Update Deployment File') {
            steps {
                sh '''
                sed -i "s|image:.*|image: $DOCKER_IMAGE|g" deployment.yaml
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
            echo 'SUCCESS 🚀 Deployment completed'
        }
        failure {
            echo 'FAILED ❌ Check logs'
        }
    }
}
