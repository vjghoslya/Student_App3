# Student Management System (Flask + Pytest + Jenkins + Docker)

# 📦 CI/CD Pipeline for Flask Application using Jenkins

## 📌 Project Overview

This project implements a **CI/CD pipeline** for a Python Flask web application using **Jenkins**, **Docker**, and **GitHub**. It automates the process of building, testing, containerizing, and deploying the application to a staging server, along with email notifications for success or failure.

---

## 🎯 Objective

- Set up a Jenkins pipeline that:
  - Clones the repository
  - Creates and uses a virtual environment
  - Installs dependencies
  - Runs unit tests
  - Builds and pushes a Docker image
  - Deploys the image to a remote staging server
  - Sends email notifications on build status

---

## 🧰 Prerequisites

### Jenkins Setup
- Jenkins installed on a VM or cloud instance
- Install the following Jenkins plugins:
  - **Pipeline**
  - **GitHub Integration**
  - **Email Extension**
  - **Docker Pipeline**
  - **SSH Agent Plugin**

### Jenkins Environment Configuration
- Python 3.11+ installed
- Docker installed and running
- Git installed
- SMTP configured in Jenkins for email notifications
- Credentials added:
  - **docker-hub-credentials** (username/password)
  - **student-node** (SSH private key for staging server)

---

## 🗃️ Source Code

- Fork the sample Python Flask app:
  - 🔗 [https://github.com/vjghoslya/studentapp3](https://github.com/vjghoslya/studentapp3)

- Clone it on the Jenkins server or allow Jenkins to clone it during the pipeline execution.

---

## 🛠️ Jenkinsfile Overview

Below are the stages defined in the `Jenkinsfile`:

### 1. **Clone Repo**
- Clones the `main` branch of the app repository.

### 2. **Install Dependencies in Virtualenv**
- Creates a virtual environment
- Installs dependencies from `requirements.txt`

### 3. **Run Tests**
- Executes unit tests using `pytest`
- Fails the pipeline if any test fails

### 4. **Dockerize Application**
- Builds a Docker image `studentapp-image:latest`
- Tags and pushes it to Docker Hub using credentials

### 5. **Deploy to Staging via SSH**
- Connects to a remote server (e.g., `192.168.188.142`)
- Pulls the latest Docker image
- Stops/removes old containers
- Starts a new container exposing port `5000`

### 6. **Email Notifications**
- Sends emails to notify build **success** or **failure** using `post` block

---

## 🐳 Dockerfile

Here’s the `Dockerfile` used to containerize the Flask app:

```dockerfile
FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app
COPY . .

RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 5000
CMD ["python", "app.py"]
