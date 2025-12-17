

# 🔐 LAB: Build Docker Image and Scan Using Trivy in Jenkins

---

## 🎯 Lab Objective

By completing this lab, you will be able to:

* Create a Dockerfile for an application
* Build a Docker image using Jenkins Pipeline
* Scan the Docker image using **Trivy**
* Disable secret scanning for performance
* Handle real-world CI issues like disk exhaustion
* Apply best practices used in production pipelines

---

## 🧰 Prerequisites

### Infrastructure

* Jenkins server (EC2 / VM / Local)
* Minimum **20 GB disk** recommended
* Ubuntu 20.04+

### Software

* Jenkins installed and running
* Docker installed
* Trivy installed
* Git installed

---

## 🟢 STEP 1: Prepare Jenkins Server

## Install Java
```
sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version
```
## Install Jenkins

```
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins
```


### 1.1 Install Docker

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

### 1.2 Allow Jenkins to Run Docker

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

---

## 🟢 STEP 2: Install Trivy

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -

echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" \
| sudo tee /etc/apt/sources.list.d/trivy.list

sudo apt update
sudo apt install trivy -y
```

### Verify

```bash
trivy --version
```

---

## 🟢 STEP 3: Fork the Git Repository

### 3.1 Repository Structure

```
DockerFile-build-and-scan-with-Trivy---JenkinsPipeline/
├── app/
│   ├── app.js
│   └── package.json
├── Dockerfile
└── Jenkinsfile
```

## 🟢 STEP 7: Create Jenkins Pipeline Job

1. Jenkins Dashboard → **New Item**
2. Job name: `docker-trivy-lab`
3. Select **Pipeline**
4. Click **OK**

---

## 🟢 STEP 8: Jenkinsfile (Final & Stable)

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = "trivy-demo"
        IMAGE_TAG  = "1.0"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/bhuvan-raj/DockerFile-build-and-scan-with-Trivy---JenkinsPipeline.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                trivy image \
                  --scanners vuln \
                  --severity HIGH,CRITICAL \
                  $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        always {
            sh 'docker images'
        }
    }
}
```

---



## 🟢 STEP 10: (Optional) Fail Pipeline on Critical Issues

```bash
trivy image \
  --scanners vuln \
  --severity CRITICAL \
  --exit-code 1 \
  trivy-demo:1.0
```

---

## 🟢 STEP 11: Verification

```bash
docker images
df -h
```

---

