# **🔵 LAB: Build & List a Docker Image Using Jenkins Pipeline**

---

# **1. PREREQUISITES**

Ensure the following exist:

* Jenkins installed (Ubuntu/CentOS/EC2)
* Docker installed on the same Jenkins host
* Jenkins user must be allowed to run Docker commands

---

# **2. INSTALL REQUIRED JENKINS PLUGINS**

Go to:

**Manage Jenkins → Plugins → Available Plugins**

Install:

1. **Pipeline**
2. **Docker Pipeline**
3. **Git Plugin**

Then restart Jenkins.

---

# **3. CONFIGURE DOCKER PERMISSIONS FOR JENKINS**

This is the MOST important part.

### **Step 1 — Add Jenkins to Docker group**

Run on server:

```bash
sudo usermod -aG docker jenkins
```

### **Step 2 — Restart services**

```bash
sudo systemctl restart docker
sudo systemctl restart jenkins
```

### **Step 3 — Verify as Jenkins user**

```bash
sudo su - jenkins
docker ps
```

✔ If this works → Jenkins can use Docker
❌ If permission denied appears → fix socket permissions:

```bash
sudo chmod 666 /var/run/docker.sock
```

---

# **4. PREPARE GITHUB REPOSITORY**

Create a GitHub repo named (example):

```
docker-jenkins-demo
```

### **Add file: Dockerfile**

```dockerfile
FROM alpine:latest
CMD ["echo", "Hello from Jenkins Docker Build"]
```

### **Add file: Jenkinsfile**

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = "demo-image"
        IMAGE_TAG  = "v1"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh """
                    echo "Building Docker Image..."
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('List Docker Images') {
            steps {
                sh """
                    echo "Listing Docker Images..."
                    docker images
                """
            }
        }
    }
}
```

Commit and push both files.

---

# **5. CREATE PIPELINE JOB IN JENKINS**

### **Step-by-step**

1. Open Jenkins UI
2. Click **New Item**
3. Name:

   ```
   docker-pipeline
   ```
4. Select **Pipeline**
5. Click **OK**
6. Scroll to **Pipeline** section
7. Choose:
   **Pipeline script from SCM**
8. SCM → **Git**
9. Repo URL:

   ```
   https://github.com/bhuvan-raj/docker-jenkins-demo.git
   ```
10. Branch:

```
main
```

11. Script path:

```
Jenkinsfile
```

12. Save

---

# **6. RUN THE PIPELINE**

Click **Build Now**

It will:

### ✔ Check out your GitHub repo

Includes Dockerfile & Jenkinsfile

### ✔ Build Docker image

Command executed:

```
docker build -t demo-image:v1 .
```

### ✔ List Docker images

Command executed:

```
docker images
```

---

# **7. VERIFY OUTPUT**

### On Jenkins console:

You should see:

```
Building Docker Image...
Successfully built <image-id>
Listing Docker Images...
REPOSITORY         TAG     IMAGE ID      SIZE
demo-image         v1      <image-id>    7MB
```

### On Jenkins host (optional):

SSH into server and run:

```bash
docker images | grep demo-image
```

---

# **🎉 LAB SUCCESSFULLY COMPLETED**

You now have a fully working lab where Jenkins:

* Checks out your GitHub repo
* Builds your Docker image
* Lists all local Docker images
* Does NOT push to Docker Hub
* Uses only local Docker engine

---
