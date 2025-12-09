# **LAB: Build and Push a Docker Image to Docker Hub Using Jenkins Pipeline**

---

## **1. Prerequisites**

### **On Your Machine**

1. Install **Docker Desktop** (Windows).
2. Install **Git**.
3. Ensure your system supports **WSL2** (Docker Desktop requires it).

### **On Your Jenkins Server (Linux recommended)**

1. Jenkins installed and running.
2. Docker installed on the Jenkins server.
3. Jenkins user added to the Docker group:

   ```bash
   sudo usermod -aG docker jenkins
   sudo systemctl restart jenkins
   ```
4. Jenkins plugins required:

   * **Pipeline**
   * **Credentials Binding**
   * **Docker Pipeline**

---

## **2. Create a Docker Hub Account**

1. Visit Docker Hub.
2. Create a new repository:

   * Name: `jenkins-demo`
   * Visibility: Public (recommended for lab)

---

## **3. Add Docker Hub Credentials in Jenkins**

1. Go to **Jenkins Dashboard → Manage Jenkins → Credentials → System → Global**.
2. Add a new credential:

   * **Kind**: Username & password
   * **ID**: `dockerhub-creds`
   * **Username**: Your Docker Hub username
   * **Password**: Docker Hub password

---

## **4. Create a Sample Application for the Docker Image**

### **On Windows**

```bash
mkdir jenkins-docker-demo
cd jenkins-docker-demo
```

Create an `app.py`:

```python
print("Hello from Jenkins Docker Pipeline!")
```

Create a `Dockerfile`:

```dockerfile
FROM python:3.9-slim
COPY app.py /app.py
CMD ["python", "/app.py"]
```


## **5. Create a Pipeline Job in Jenkins**

### **Step-by-step**

1. Go to **Jenkins Dashboard**
2. Click **New Item**
3. Enter name: `docker-build-push`
4. Select **Pipeline** → OK
5. Scroll down to the **Pipeline** section
6. Select **Pipeline script from SCM**
7. SCM: **Git**
8. Repository URL:

   ```
   https://github.com/YOUR-ID/jenkins-docker-demo.git
   ```
9. Script Path:

   ```
   Jenkinsfile
   ```

---

## **6. Create the Jenkinsfile in Your Repository**

Create a file named **Jenkinsfile** in the repo:

```groovy
pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_NAME = "YOUR_DOCKERHUB_USERNAME/jenkins-demo"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -t ${IMAGE_NAME}:latest .
                    """
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    sh """
                    echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin
                    """
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    sh """
                    docker push ${IMAGE_NAME}:latest
                    """
                }
            }
        }
    }

    post {
        always {
            sh "docker logout"
        }
    }
}
```

---

## **7. Run the Pipeline**

1. Click the pipeline job: `docker-build-push`.
2. Select **Build Now**.
3. Observe:

   * Build stage: Docker image creation
   * Login stage
   * Push to Docker Hub

---

## **8. Verify the Image in Docker Hub**

1. Go to Docker Hub.
2. Open your repository `jenkins-demo`.
3. You should see the **latest** tag pushed by Jenkins.

---

## **9. Test the Image on Your Windows System**

Run:

```bash
docker pull YOUR_DOCKERHUB_USERNAME/jenkins-demo:latest
docker run --rm YOUR_DOCKERHUB_USERNAME/jenkins-demo:latest
```

Expected output:

```
Hello from Jenkins Docker Pipeline!
```

---

