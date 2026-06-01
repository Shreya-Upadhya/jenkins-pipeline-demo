pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'slayerass/jenkins-pipeline-demo'
        DOCKER_TAG = 'latest'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '📥 Pulling code from GitHub...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo '📦 Installing npm packages...'
                bat 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Running tests...'
                bat 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                bat "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo '📤 Pushing to Docker Hub...'
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat '''
                        echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                        docker push %DOCKER_IMAGE%:%DOCKER_TAG%
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                echo '🚀 Deploying application...'
                bat '''
                    docker stop jenkins-demo-app || exit 0
                    docker rm jenkins-demo-app || exit 0
                    docker run -d --name jenkins-demo-app -p 3000:3000 %DOCKER_IMAGE%:%DOCKER_TAG%
                '''
                echo '✅ Deployment successful!'
                echo '📍 App running at: http://localhost:3000'
            }
        }
    }

    post {
        success {
            echo '🎉 Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check console output.'
        }
    }
}
