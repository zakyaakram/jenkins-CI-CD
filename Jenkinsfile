pipeline {
    agent any

    environment {
        IMAGE_NAME     = "zakyaakram52/jenkins-app"
        IMAGE_TAG      = "${BUILD_NUMBER}"
        FULL_IMAGE     = "${IMAGE_NAME}:${IMAGE_TAG}"
        KUBE_NAMESPACE = "ivolve"
    }

    stages {

        // 🔥 Strong cleanup (fixes your main issue)
        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        // 🔍 Checkout code
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/Ibrahim-Adel15/Jenkins_App.git'
            }
        }

        // 🧪 Debug (VERY IMPORTANT for failures)
        stage('Debug Workspace') {
            steps {
                sh '''
                echo "Current directory:"
                pwd
                echo "Files:"
                ls -la
                '''
            }
        }

        // ⚙️ Verify tools (prevents silent failures)
        stage('Check Tools') {
            steps {
                sh '''
                java -version
                mvn -v
                '''
            }
        }

        // 🧹 Fix Maven corruption issues
        stage('Reset Maven Cache') {
            steps {
                sh 'rm -rf ~/.m2/repository || true'
            }
        }

        // 1️⃣ Unit Test
        stage('Run Unit Tests') {
            steps {
                sh 'mvn clean test'
            }
        }

        // 2️⃣ Build App
        stage('Build Application') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        // 🐳 Clean Docker (prevents disk issues)
        stage('Docker Cleanup') {
            steps {
                sh 'docker system prune -af || true'
            }
        }

        // 3️⃣ Build Docker Image
        stage('Build Docker Image') {
            steps {
                sh '''
                echo "Building image: $FULL_IMAGE"
                docker build --no-cache -t $FULL_IMAGE .
                '''
            }
        }

        // 4️⃣ Push to Docker Hub
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

        // 5️⃣ Delete Local Image
        stage('Delete Local Image') {
            steps {
                sh 'docker rmi $FULL_IMAGE || true'
            }
        }

        // 6️⃣ Update deployment.yaml
        stage('Update Deployment File') {
            steps {
                sh '''
                sed -i "s|image: .*|image: ${FULL_IMAGE}|g" deployment.yaml
                '''
            }
        }

        // 7️⃣ Deploy to Kubernetes
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
            echo "FAILED: Check logs ❌"
        }
    }
}
