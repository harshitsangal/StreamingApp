pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '704593926034'
        ECR_BASE = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/harshitsangal/StreamingApp.git'
            }
        }

        stage('Login to AWS ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials'
                ]]) {
                    sh '''
                    echo "Logging into AWS ECR..."
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ECR_BASE
                    '''
                }
            }
        }

        stage('Build & Push Images') {
            steps {
                script {

                    def services = [
                        [name: "auth", path: "./backend/authService"],
                        [name: "admin", path: "./backend/adminService"],
                        [name: "chat", path: "./backend/chatService"],
                        [name: "streaming", path: "./backend/streamingService"],
                        [name: "frontend", path: "./frontend"]
                    ]

                    for (service in services) {

                        def imageName = "streamingapp/${service.name}"
                        def fullImage = "${ECR_BASE}/${imageName}:${IMAGE_TAG}"

                        sh """
                        echo "=============================="
                        echo "Building ${service.name}..."
                        docker build -t ${service.name}:${IMAGE_TAG} ${service.path}

                        echo "Tagging ${service.name}..."
                        docker tag ${service.name}:${IMAGE_TAG} ${fullImage}

                        echo "Pushing ${service.name}..."
                        docker push ${fullImage}
                        echo "=============================="
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ All services built and pushed to ECR successfully!'
        }
        failure {
            echo '❌ Build failed. Check logs carefully.'
        }
    }
}