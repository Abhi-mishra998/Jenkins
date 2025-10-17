

# Jenkins CI/CD Pipeline with Docker & Email Notifications

This project demonstrates a **complete CI/CD pipeline using Jenkins, Docker, GitHub, AWS EC2, and email notifications**. Whenever any event occurs in the repository (like a push), the pipeline will automatically build, push the Docker image, and notify via email.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Create a Gmail Email & App Password](#create-a-gmail-email--app-password)
3. [Launch an EC2 Instance](#launch-an-ec2-instance)
4. [Install Jenkins on EC2](#install-jenkins-on-ec2)
5. [Setup Docker](#setup-docker)
6. [Setup GitHub Webhook](#setup-github-webhook)
7. [Configure Jenkins Pipeline](#configure-jenkins-pipeline)
8. [Email Notification Setup](#email-notification-setup)
9. [Testing the Pipeline](#testing-the-pipeline)

---

## Prerequisites

* AWS account with permission to create EC2 instances.
* GitHub repository for your project.
* Gmail account for sending build notifications.
* Basic knowledge of Linux commands.

---

## Create a Gmail Email & App Password

1. Create a new Gmail email (e.g., `myci.cdpipeline@gmail.com`).
2. Go to [Google Account Security](https://myaccount.google.com/security) → **Two-Step Verification** → turn it ON.
3. Navigate to [App Passwords](https://myaccount.google.com/apppasswords) → generate a new **App Password** for “Mail”.
4. Keep the email and password handy for Jenkins SMTP configuration.

---

## Launch an EC2 Instance

Use the following AWS CLI command to create a **t2.medium EC2 instance**:

```bash
aws ec2 run-instances \
  --image-id ami-02d26659fd82cf299 \
  --count 1 \
  --instance-type t2.medium \
  --security-group-ids sg-02602e6dba9bd4b7d \
  --subnet-id subnet-07d6f1903894d6e06 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=My-T2Medium-Instance}]'
```

* **AMI ID**: Ubuntu Server
* **Security Group**: Allows HTTP, HTTPS, and SSH access
* **Subnet ID**: Your VPC subnet

---

## Install Jenkins on EC2

1. SSH into your EC2 instance:

```bash
ssh -i "your-key.pem" ubuntu@<EC2-Public-IP>
```

2. Install Jenkins:

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

3. Access Jenkins in your browser: `http://<EC2-Public-IP>:8080`

---

## Setup Docker

1. Install Docker on EC2:

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu
```

2. Verify Docker:

```bash
docker --version
```

---

## Setup GitHub Webhook

1. Open your GitHub repository → **Settings → Webhooks → Add webhook**
2. **Payload URL**: `http://<EC2-Public-IP>:8080/github-webhook/`
3. **Content type**: `application/json`
4. **Events**: Select "Just the push event"
5. Save the webhook

---

## Configure Jenkins Pipeline

1. Install **GitHub Integration Plugin** in Jenkins.
2. Install **Email Extension Plugin** for email notifications.
3. Create a new **Pipeline Job** in Jenkins and paste this Jenkinsfile:

```groovy
pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        IMAGE_NAME = "abhishek8056/jenkins-repo"
        NOTIFY_EMAIL = "testingabhi9896@gmail.com"
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo "Cloning repository..."
                git branch: 'main', url: 'https://github.com/Abhi-mishra998/Jenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Login to DockerHub') {
            steps {
                echo "Logging into DockerHub..."
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Pushing Docker image to DockerHub..."
                sh 'docker push $IMAGE_NAME:latest'
            }
        }
    }

    post {
        always { cleanWs() }

        success {
            emailext(
                to: "${NOTIFY_EMAIL}",
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build succeeded! Job: ${env.JOB_NAME}, Build Number: ${env.BUILD_NUMBER}, Docker Image: ${IMAGE_NAME}:latest"
            )
        }

        failure {
            emailext(
                to: "${NOTIFY_EMAIL}",
                subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build failed! Job: ${env.JOB_NAME}, Build Number: ${env.BUILD_NUMBER}"
            )
        }
    }
}
```

---

## Email Notification Setup

1. Go to **Manage Jenkins → Configure System → Extended E-mail Notification**
2. SMTP settings:

* SMTP server: `smtp.gmail.com`
* Port: `465`
* Use SSL: Yes
* Username: Your Gmail email
* Password: Your App Password

3. Test by sending a test email.

---

## Testing the Pipeline

1. Push a change to your GitHub repository.
2. Webhook triggers Jenkins automatically.
3. Jenkins will:

* Clone the repo
* Build the Docker image
* Login & push to DockerHub
* Send email notification for success/failure

---

✅ **Congratulations!** You now have a fully automated **CI/CD pipeline with Docker, Jenkins, and email notifications**.



Do you want me to prepare that?
