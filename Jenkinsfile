// =========================================================================
// Jenkins Pipeline: Build and Push Docker Image + Email Notification
// Author: Abhishek Mishra
// =========================================================================

pipeline {
    agent any

    environment {
        // DockerHub credentials (configured in Jenkins Credentials Manager)
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')

        // Docker image name
        IMAGE_NAME = "abhishek8056/jenkins-repo"

        // Email for build notifications
        NOTIFY_EMAIL = "testingabhi9896@gmail.com"
    }

    options {
        // Show timestamps in console output
        timestamps()

        // Prevent multiple builds from running at the same time
        disableConcurrentBuilds()

        // Keep only the last 10 build logs to save space
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        // Stage 1: Clone the GitHub repository
        stage('Clone Repository') {
            steps {
                echo "Cloning repository..."
                git branch: 'main', url: 'https://github.com/Abhi-mishra998/Jenkins.git'
            }
        }

        // Stage 2: Build Docker image
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        // Stage 3: Login to DockerHub
        stage('Login to DockerHub') {
            steps {
                echo "Logging in to DockerHub..."
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        // Stage 4: Push Docker image to DockerHub
        stage('Push Docker Image') {
            steps {
                echo "Pushing Docker image to DockerHub..."
                sh 'docker push $IMAGE_NAME:latest'
            }
        }
    }

    post {

        // Always clean workspace after build
        always {
            echo "Cleaning workspace..."
            cleanWs()
        }

        // On build success
        success {
            echo "Build succeeded."
            emailext(
                to: "${NOTIFY_EMAIL}",
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Build Status: SUCCESS

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Docker Image: ${IMAGE_NAME}:latest
Repository: https://github.com/Abhi-mishra998/Jenkins
Build URL: ${env.BUILD_URL}

-- Jenkins CI/CD Pipeline Notification
""",
                mimeType: 'text/plain'
            )
        }

        // On build failure
        failure {
            echo "Build failed."
            emailext(
                to: "${NOTIFY_EMAIL}",
                subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Build Status: FAILURE

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Repository: https://github.com/Abhi-mishra998/Jenkins
Build URL: ${env.BUILD_URL}

-- Jenkins CI/CD Pipeline Notification
""",
                mimeType: 'text/plain'
            )
        }

        // On build unstable (optional)
        unstable {
            echo "Build unstable."
            emailext(
                to: "${NOTIFY_EMAIL}",
                subject: "UNSTABLE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Build Status: UNSTABLE

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}
Check Jenkins console for warnings:
${env.BUILD_URL}

-- Jenkins CI/CD Pipeline Notification
""",
                mimeType: 'text/plain'
            )
        }

        // On build aborted manually (optional)
        aborted {
            echo "Build aborted by user."
            emailext(
                to: "${NOTIFY_EMAIL}",
                subject: "ABORTED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Build Status: ABORTED

Job: ${env.JOB_NAME}
Build Number: ${env.BUILD_NUMBER}

The build was manually aborted by a user.

-- Jenkins CI/CD Pipeline Notification
""",
                mimeType: 'text/plain'
            )
        }
    }
}
