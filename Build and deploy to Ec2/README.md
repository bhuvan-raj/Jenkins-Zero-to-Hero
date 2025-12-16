
---

# 🧪 LAB: Deploy Java (Spring Boot) Application to EC2 Using Jenkins Pipeline

---

## 🎯 Lab Objective

* Build a Java application using **Maven**
* Automate CI/CD using **Jenkins Declarative Pipeline**
* Deploy the packaged JAR to an **Ubuntu EC2 instance**
* Start the application remotely via **SSH**

---

## 🧰 Tools & Technologies

| Component        | Purpose                   |
| ---------------- | ------------------------- |
| Jenkins          | CI/CD automation          |
| GitHub           | Source code repository    |
| Maven            | Build tool                |
| Java 17          | Runtime                   |
| AWS EC2 (Ubuntu) | Deployment server         |
| SSH Agent Plugin | Secure SSH authentication |

---

## 📌 Prerequisites

### Jenkins Server

* Jenkins installed and running
* Internet access enabled

### EC2 Instance

* Ubuntu 22.04 / 24.04
* Port **22** and **8080** open in Security Group
* Java 17 installed

---

## 🔹 STEP 1: Launch Ubuntu EC2 Instance

1. Create EC2 instance

   * AMI: Ubuntu 22.04 / 24.04
   * Instance type: t2.micro
   * Key pair: Create or use existing `.pem`
2. Security Group rules:

   * SSH → 22 → Anywhere
   * HTTP/App → 8080 → Anywhere

---

## 🔹 STEP 2: Prepare EC2 Server

### Connect to EC2

```bash
ssh -i ec2-key.pem ubuntu@<EC2_PUBLIC_IP>
```

### Install Java

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
java -version
```

### Create application directory

```bash
sudo mkdir -p /opt/app
sudo chown ubuntu:ubuntu /opt/app
```

---

## 🔹 STEP 3: Install Jenkins Plugins

Go to **Manage Jenkins → Plugins** and install:

* SSH Agent Plugin
* Pipeline
* Git
* Maven Integration

Restart Jenkins if required.

---

## 🔹 STEP 4: Configure Jenkins Tools

### Maven

```
Manage Jenkins → Global Tool Configuration → Maven
Name: maven-3.9
```

### JDK

```
Manage Jenkins → Global Tool Configuration → JDK
Name: JDK17
```

---

## 🔹 STEP 5: Add EC2 SSH Key to Jenkins

1. Go to **Manage Jenkins → Credentials**
2. Add:

   * Kind: SSH Username with private key
   * Username: `ubuntu`
   * Private Key: paste EC2 `.pem`
   * ID: `ec2-ssh-key`

---

## 🔹 STEP 6: Create GitHub Repository

Repository should contain:

```
pom.xml
src/main/java/com/example/demo/DemoApplication.java
Jenkinsfile
```

Use your repo:

```
https://github.com/bhuvan-raj/MavenApp-deployment-to-Ec2-with-JenkinsPipeline.git
```

---

## 🔹 STEP 7: Final Working Jenkinsfile

```groovy
pipeline {
    agent any

    tools {
        maven 'maven-3.9'
        jdk 'JDK17'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/bhuvan-raj/MavenApp-deployment-to-Ec2-with-JenkinsPipeline.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh '''
                        echo "Copying artifact to EC2..."
                        scp -o StrictHostKeyChecking=no \
                            target/demo-1.0.0.jar \
                            ubuntu@<EC2_PUBLIC_IP>:/opt/app/

                        echo "Starting application on EC2..."
                        ssh -T -o StrictHostKeyChecking=no ubuntu@<EC2_PUBLIC_IP> '
                            pkill -f demo-1.0.0.jar || true
                            setsid nohup java -jar /opt/app/demo-1.0.0.jar \
                                > /opt/app/app.log 2>&1 < /dev/null &
                            exit 0
                        '
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully."
        }
        failure {
            echo "Deployment failed. Check Jenkins or EC2 logs."
        }
    }
}
```

⚠️ Replace `<EC2_PUBLIC_IP>` with your actual EC2 public IP.

---

## 🔹 STEP 8: Create Jenkins Pipeline Job

1. New Item → Pipeline
2. Pipeline Definition → Pipeline script from SCM
3. SCM → Git
4. Repository URL → GitHub repo
5. Branch → main
6. Save

---

## 🔹 STEP 9: Build the Pipeline

Click **Build Now**

### Expected Output

* Maven build succeeds
* JAR copied to EC2
* Application started in background
* Jenkins build shows **SUCCESS**

---

## 🔹 STEP 10: Verify Deployment

### On EC2

```bash
ps -ef | grep demo-1.0.0.jar
tail -f /opt/app/app.log
```

### From browser

```
http://<EC2_PUBLIC_IP>:8080
```

---

## 🧠 Common Errors & Fixes

| Error              | Cause                    | Fix                      |
| ------------------ | ------------------------ | ------------------------ |
| sshagent not found | Plugin missing           | Install SSH Agent Plugin |
| exit code 255      | SSH session not detached | Use `ssh -T + setsid`    |
| exit code 143      | Jenkins SIGTERM          | Disable TTY + exit 0     |
| Permission denied  | /opt/app ownership       | chown ubuntu             |

---

