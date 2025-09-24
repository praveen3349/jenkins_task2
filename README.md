## Overview

This project demonstrates a **simple Jenkins CI/CD pipeline** for a **Python Flask web application** using **Docker**.  

The pipeline automates:

1. Pulling code from GitHub
2. Building a Docker image
3. Pushing the Docker image to Docker Hub

The Flask app is a basic web server that displays a simple welcome message.

---

## Features

- Python Flask web application  
- Dockerized for container deployment  
- Automated Jenkins CI/CD pipeline  
- Pushes Docker image to Docker Hub  
- Pipeline triggers automatically on GitHub commits  

---

## Project Structure

**flask-demo-app/**
- app.py # Flask application
- requirements.txt # Dependencies
- Dockerfile # Docker instructions
- Jenkinsfile # Jenkins pipeline script
- images/ # Screenshots for README
- README.md # This file

---

## Step 1: Install and Run Locally

1. **Clone the repository**

git clone https://github.com/praveen3349/jenkins_task2.git
cd jenkins_task2
Build the Docker image

docker build -t flask-demo-app:latest .
Run the Docker container

docker run -d -p 5000:5000 --name flask-app flask-demo-app:latest
Open in browser

http://localhost:5000
<!-- Replace with actual screenshot -->

## Step 2: Jenkins Setup

Running Jenkins in Docker Desktop

docker run -d \
  -p 8080:8080 \
  -p 50000:50000 \
  --name jenkins \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts
  
Open http://localhost:8080 in your browser

Install Pipeline and Docker Pipeline plugins

## Step 3: Add Docker Hub Credentials in Jenkins

- Jenkins → Manage Jenkins → Credentials → Global → Add Credentials

- Type: Username & Password

- Username → Docker Hub username

- Password → Docker Hub password or access token

- ID → dockerhub-creds (used in Jenkinsfile)

## Step 4: Jenkins Pipeline (Jenkinsfile)
Add this Jenkinsfile in the root of your repo:


    pipeline {
    agent any
    environment {
        DOCKER_HUB_CREDENTIALS = 'dockerhub-creds'
        IMAGE_NAME = 'flask-demo-app'
        DOCKER_USERNAME = 'YOUR_DOCKER_USERNAME'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/praveen3349/jenkins_task2.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_USERNAME/$IMAGE_NAME:latest ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "$DOCKER_HUB_CREDENTIALS",
                                                 usernameVariable: 'USERNAME',
                                                 passwordVariable: 'PASSWORD')]) {
                    sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"
                    sh "docker push $DOCKER_USERNAME/$IMAGE_NAME:latest"
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed!'
        }
    }


Notes:

Replace YOUR_DOCKER_USERNAME with your Docker Hub username.

This pipeline builds and pushes the Docker image to Docker Hub.

## Step 5: Configure Jenkins Pipeline

Jenkins → New Item → Pipeline → Name: flask-demo-pipeline

Definition → Pipeline script from SCM

SCM → Git → Repository URL: https://github.com/praveen3349/jenkins_task2.git

Branch → main

Script Path → Jenkinsfile


## Step 6: GitHub Webhook (Optional for Auto-trigger)

GitHub → Repository Settings → Webhooks → Add webhook

Payload URL: http://<JENKINS_URL>/github-webhook/

Content type: application/json

Trigger → Just the push event

Now every push to main triggers the Jenkins pipeline automatically.

## Step 7: Run & Test Pipeline
Push a commit to GitHub:

git add .
git commit -m "Test Jenkins pipeline"
git push

Jenkins → Dashboard → flask-demo-pipeline → Build Now

Check logs for each stage: Checkout → Build Docker → Push Docker

Docker image will appear in Docker Hub
