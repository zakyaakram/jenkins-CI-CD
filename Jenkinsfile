pipeline {
    agent any

    environment {
        DOCKER_IMAGE   = "zakyaakram52/jenkins-app:latest"
        KUBE_NAMESPACE = "ivolve"
    }

    stages {

        stage('Checkout') {
            steps {
                cleanWs()  // clears workspace before cloning
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {
                    sh '''
                    git config --global credential.helper store
                    git clone https://$GIT_USER:$GIT_PASS@github.com/zakyaakram/jenkins-CI-CD.git
                    '''
                }
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
                docker version
                docker build -t $DOCKER_IMAGE .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
