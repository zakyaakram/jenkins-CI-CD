pipeline {
    agent any

    environment {
        IMAGE_NAME     = "zakyaakram52/jenkins-app"
        IMAGE_TAG      = "${BUILD_NUMBER}"
        FULL_IMAGE     = "${IMAGE_NAME}:${IMAGE_TAG}"
        KUBE_NAMESPACE = "ivolve"
    }

    stages {

        // 1. Clone Repo
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/Ibrahim-Adel15/Jenkins_App.git'
            }
        }

        // 2. Unit Test
        stage('Run Unit Tests') {
            steps {
                sh 'mvn clean test'
            }
        }

        // 3. Build App
        stage('Build Application') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        // 4. Build Docker Image
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $FULL_IMAGE .'
            }
        }

        // 5. Push to Docker Hub
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $FULL_IMAGE
                    '''
                }
            }
        }

        // 6. Delete Local Image
        stage('Delete Local Image') {
            steps {
                sh 'docker rmi $FULL_IMAGE || true'
            }
        }

        // 7. Update deployment.yaml
        stage('Update Deployment File') {
            steps {
                sh '''
                sed -i "s|image: .*|image: ${FULL_IMAGE}|g" deployment.yaml
                '''
            }
        }

        // 8. Deploy to Kubernetes
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([string(
                    credentialsId: 'kube-config',
                    variable: 'KUBECONFIG_CONTENT'
                )]) {
                    writeFile file: 'kubeconfig.yaml', text: "$KUBECONFIG_CONTENT"

                    sh '''
                    kubectl --kubeconfig=kubeconfig.yaml apply -f deployment.yaml -n $KUBE_NAMESPACE
                    kubectl --kubeconfig=kubeconfig.yaml get pods -n $KUBE_NAMESPACE
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'rm -f kubeconfig.yaml || true'
            echo "Pipeline finished"
        }

        success {
            echo "SUCCESS: Deployment completed 🚀"
        }

        failure {
            echo "FAILED: Something went wrong ❌"
        }
    }
}
