pipeline {
    agent any

    environment {
        DOCKER_IMAGE   = "zakyaakram52/jenkins-app:${BUILD_NUMBER}"
        KUBE_NAMESPACE = "ivolve"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                git credentialsId: 'github-creds',
                    url: 'https://github.com/zakyaakram/jenkins-CI-CD.git'
            }
        }

        stage('Debug Workspace') {
            steps {
                sh '''
                echo "=== WORKSPACE FILES ==="
                pwd
                ls -la
                '''
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn -v'
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

        stage('Update Deployment File') {
            steps {
                sh '''
                sed -i "s|image: .*|image: ${DOCKER_IMAGE}|g" deployment.yaml
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([string(credentialsId: 'kube-config', variable: 'KUBECONFIG_CONTENT')]) {
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
            echo "Deployment SUCCESS 🚀"
        }

        failure {
            echo "Pipeline FAILED ❌"
        }
    }
}
