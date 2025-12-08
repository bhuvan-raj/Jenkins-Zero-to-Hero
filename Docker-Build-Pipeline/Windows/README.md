# **🔵 LAB: Build & List a Docker Image Using Jenkins Pipeline (Windows Version)**

---

# **1. INSTALL REQUIRED TOOLS ON WINDOWS**

## **1️⃣ Install Docker Desktop**

Download → Install → Enable:

* **WSL2 backend**
* **Expose daemon on tcp://localhost:2375 without TLS** (important!)

To enable this:

1. Open Docker Desktop
2. Go to **Settings → General**
3. Enable:
   ✔ *Expose daemon on tcp://localhost:2375 without TLS*

This will allow Jenkins to use Docker without needing admin rights.

---

## **2️⃣ Install Jenkins for Windows**

If not installed:

1. Download Jenkins MSI
2. Install as **Windows Service**
3. Default home:

   ```
   C:\Program Files (x86)\Jenkins
   ```

---

# **2. CONFIGURE DOCKER ACCESS FOR JENKINS (CRITICAL)**

On Windows, Jenkins runs as the **LocalSystem** user, which **cannot access Docker** unless configured.

To fix this:

---

## **Method 1 (Recommended): Enable Docker over TCP**

You already enabled in Docker Desktop:

✔ Expose daemon on **tcp://localhost:2375**

Now configure Jenkins global environment variables:

1. Go to Jenkins UI
2. **Manage Jenkins → Configure System**
3. Under **Global Properties**, enable environment variables
4. Add:

```
DOCKER_HOST = tcp://localhost:2375
```

This allows Jenkins to execute docker commands even without privileged access.

---

## **Method 2 (Optional): Run Jenkins as your Windows User**

If Method 1 fails:

1. Open **Services**
2. Find “Jenkins”
3. Right-click → **Properties → Log On**
4. Change from *Local System account* → to your **Windows user** (the one who uses Docker Desktop)
5. Restart Jenkins service

Now Jenkins inherits Docker permissions.

---

# **3. INSTALL REQUIRED JENKINS PLUGINS**

Go to:

**Manage Jenkins → Plugins → Available Plugins**

Install:

1. Pipeline
2. Git Plugin
3. Docker Pipeline (optional)

Restart Jenkins.

---

# **4. PREPARE GITHUB REPO**

Your GitHub repo must contain:

### **Dockerfile**

```dockerfile
FROM alpine:latest
CMD ["echo", "Hello from Jenkins Windows Docker Build"]
```

### **Jenkinsfile**

Windows version uses **powershell** instead of `sh`.

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
                powershell """
                    Write-Host 'Building Docker image...'
                    docker build -t ${env.IMAGE_NAME}:${env.IMAGE_TAG} .
                """
            }
        }

        stage('List Docker Images') {
            steps {
                powershell """
                    Write-Host 'Listing Docker images...'
                    docker images
                """
            }
        }
    }
}
```

Push the repo to GitHub.

---

# **5. CREATE PIPELINE JOB IN JENKINS (WINDOWS)**

1. Open Jenkins
2. **New Item** → Pipeline
3. Name:

   ```
   docker-windows-pipeline
   ```
4. Select **Pipeline script from SCM**
5. SCM → Git
6. Repo URL:

   ```
   https://github.com/bhuvan-raj/docker-jenkins-demo.git
   ```
7. Script path:

   ```
   Jenkinsfile
   ```
8. Save

---

# **6. RUN THE PIPELINE**

Click **Build Now**

If configured properly, you will see:

### ✔ Build Docker image

### ✔ List Docker images

### ✔ No permission denied errors

Example console output:

```
Building Docker image...
Sending build context to Docker daemon...
Successfully built <id>
Listing Docker images...
demo-image   v1    <id>
```

---

# **7. VERIFY MANUALLY ON WINDOWS**

Open **PowerShell** as your Windows user and run:

```powershell
docker images
```

You should see:

```
REPOSITORY     TAG   IMAGE ID     SIZE
demo-image     v1    abcd1234     7MB
```

---

# ** 🎉 LAB COMPLETED ON WINDOWS**

You now have a working Jenkins pipeline on Windows that:

* Uses Docker Desktop
* Builds Docker images
* Lists Docker images
* Uses Powershell steps
* Uses Windows-friendly Docker configuration

---
