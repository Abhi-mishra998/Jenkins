pipeline {
    agent any

    environment {
        // DockerHub credentials (configured in Jenkins Credentials)
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')

        // Docker image name (your repository)
        IMAGE_NAME = "abhishek8056/jenkins-repo"

        // Email for build notifications
        NOTIFY_EMAIL = "testingabhi9896@gmail.com"
    }

    options {
        // Show timestamps in console output
        timestamps()

        // Enable colored output in Jenkins logs
        ansiColor('xterm')

        // Prevent multiple builds running at the same time
        disableConcurrentBuilds()

        // Keep only the last 10 build logs to save space
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        // Step 1: Clone the repository
        stage('Clone Repository') {
            steps {
                echo "[INFO] Cloning repository..."
                git branch: 'main', url: 'https://github.com/Abhi-mishra998/Jenkins.git'
            }
        }

        // Step 2: Build the Docker image
        stage('Build Docker Image') {
            steps {
                echo "[BUILD] Building Docker image..."
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        // Step 3: Log in to DockerHub using Jenkins credentials
        stage('Login to DockerHub') {
            steps {
                echo "[LOGIN] Logging into DockerHub..."
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        // Step 4: Push Docker image to DockerHub
        stage('Push Docker Image') {
            steps {
                echo "[PUSH] Pushing Docker image to DockerHub..."
                sh 'docker push $IMAGE_NAME:latest'
            }
        }
    }

    // Post-build actions (run after all stages)
    post {

        // Always clean up workspace after build
        always {
            echo "[INFO] Cleaning workspace..."
            cleanWs()
        }

        // Send email when build succeeds
        success {
            echo "[SUCCESS] Build completed successfully."
            emailext(
                to: "${NOTIFY_EMAIL}",
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                Build Status: SUCCESS

                Job: ${env.JOB_NAME}
                Build Number: ${env.BUILD_NUMBER}
                Docker Image: ${IMAGE_NAME}:latest
                Repository: https://github.com/Abhi-mishra998/Jenkins

                View Details:
                ${env.BUILD_URL}

                -- Jenkins CI/CD Pipeline Notification
                """,
                mimeType: 'text/plain'
            )
        }

        // Send email when build fails
        failure {
            echo "[ERROR] Build failed."
            emailext(
                to: "${NOTIFY_EMAIL}",
                subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                Build Status: FAILURE

                Job: ${env.JOB_NAME}
                Build Number: ${env.BUILD_NUMBER}
                Repository: https://github.com/Abhi-mishra998/Jenkins

                Check Jenkins console for more details:
                ${env.BUILD_URL}

                -- Jenkins CI/CD Pipeline Notification
                """,
                mimeType: 'text/plain'
            )
        }

        // Send email if build is unstable (e.g., test failures)
        unstable {
            echo "[WARNING] Build marked as unstable."
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

        // Send email if build is aborted manually
        aborted {
            echo "[INFO] Build aborted by user."
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
