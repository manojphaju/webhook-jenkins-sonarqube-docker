pipeline {
    agent any

    environment {
        SONAR_URL = "http://localhost:9000"
        DOCKER_IMAGE = "manojphaju/webhook-jenkins-sonar-docker:${BUILD_NUMBER}"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out GitHub repository..."
                git branch: 'main', url: 'https://github.com/manojphaju/webhook-jenkins-sonarqube-docker.git'
            }
        }

        stage('Static Code Analysis') {
            steps {
                echo "Running SonarQube analysis..."
                withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh """
                        /opt/homebrew/bin/sonar-scanner \
                        -Dsonar.projectKey=webhook-jenkins-sonar-docker \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONAR_URL} \
                        -Dsonar.login=${SONAR_AUTH_TOKEN}
                    """
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    script {
                        echo "Building Docker image..."
                        sh "/usr/local/bin/docker build -t ${DOCKER_IMAGE} ."

                        echo "Logging in to Docker registry..."
                        sh "echo ${DOCKER_PASSWORD} | /usr/local/bin/docker login -u ${DOCKER_USERNAME} --password-stdin"

                        echo "Pushing Docker image..."
                        sh "/usr/local/bin/docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                echo "Stopping any existing container..."
                sh "/usr/local/bin/docker rm -f webhook-sonar-docker || true"

                echo "Running new container on port 8081..."
                sh "/usr/local/bin/docker run -d --name webhook-sonar-docker -p 8081:80 ${DOCKER_IMAGE}"
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs above.'
        }
    }
}
