
# Lab: Integrating SonarQube with Jenkins for Code Quality Analysis

---

## Lab Objective

By the end of this lab, you will be able to:

* Install and configure SonarQube
* Integrate SonarQube with Jenkins
* Configure SonarQube authentication in Jenkins
* Execute static code analysis as part of a Jenkins Pipeline
* View and interpret code quality reports in SonarQube

---

## Prerequisites

* Jenkins installed and running
* Java (JDK 11 or 17) installed on Jenkins server
* Docker installed (recommended for SonarQube)
* A sample Git repository (Java/Maven project preferred)
* Jenkins user with administrative privileges

---

## Lab Architecture

```
Developer → Git Repository → Jenkins → SonarQube
```

---

## Step 1: Start SonarQube Server (Docker – Recommended)

### 1.1 Pull and Run SonarQube

```bash
docker run -d \
  --name sonarqube \
  -p 9000:9000 \
  sonarqube
```

### 1.2 Verify SonarQube Access

* Open browser:
  `http://<sonarqube-server-ip>:9000`
* Default credentials:

  ```
  Username: admin
  Password: admin
  ```
* Change the password when prompted

---

## Step 2: Create a Project in SonarQube

1. Login to SonarQube
2. Click **Projects → Create Project**
3. Choose **Manually**
4. Enter:

   * Project Key: `java-demo`
   * Project Name: `Java Demo Project`
5. Click **Set Up**

---

## Step 3: Generate SonarQube Token

1. Go to **User → My Account → Security**
2. Enter token name:

   ```
   jenkins-token
   ```
3. Click **Generate**
4. Copy the token (important – it will not be shown again)

---

## Step 4: Install Required Jenkins Plugins

Go to **Jenkins → Manage Jenkins → Plugins → Available**

Install:

* **SonarQube Scanner**
* **Pipeline**
* **Git**
* **Maven Integration** (if not already installed)

Restart Jenkins if required.

---

## Step 5: Configure SonarQube Server in Jenkins

### 5.1 Add SonarQube Token Credential

1. Go to **Manage Jenkins → Credentials**
2. Choose:

   * Kind: **Secret Text**
   * Secret: `<paste-sonarqube-token>`
   * ID: `sonarqube-token`
   * Description: SonarQube Token

---

### 5.2 Configure SonarQube Server

1. Go to **Manage Jenkins → Configure System**
2. Scroll to **SonarQube Servers**
3. Click **Add SonarQube**
4. Configure:

   * Name: `SonarQube`
   * Server URL: `http://<sonarqube-ip>:9000`
   * Server Authentication Token: `sonarqube-token`
5. Save

---

## Step 6: Configure SonarQube Scanner Tool

1. Go to **Manage Jenkins → Global Tool Configuration**
2. Scroll to **SonarQube Scanner**
3. Click **Add SonarQube Scanner**
4. Set:

   * Name: `SonarScanner`
   * Install automatically: ✔
5. Save

---

## Step 7: Prepare Sample Maven Project

### pom.xml (Ensure this exists)

```xml
<properties>
  <sonar.projectKey>java-demo</sonar.projectKey>
  <sonar.projectName>Java Demo Project</sonar.projectName>
  <sonar.host.url>http://sonarqube:9000</sonar.host.url>
</properties>
```

> Note: Project key must match the SonarQube project key.

---

## Step 8: Create Jenkins Pipeline Job

1. Go to **New Item**
2. Select **Pipeline**
3. Name: `sonarqube-analysis`
4. Click **OK**

---

## Step 9: Jenkinsfile (SonarQube Integration)

### Declarative Pipeline

```groovy
pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        SONAR_SCANNER_HOME = tool 'SonarScanner'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/<your-repo>/java-demo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=java-demo
                    '''
                }
            }
        }
    }
}
```

Save the pipeline.

---

## Step 10: Run the Pipeline

1. Click **Build Now**
2. Observe:

   * Maven build success
   * SonarQube analysis logs
3. Ensure no authentication errors occur

---

## Step 11: Verify Results in SonarQube

1. Open SonarQube UI
2. Go to **Projects → Java Demo Project**
3. Review:

   * Bugs
   * Vulnerabilities
   * Code Smells
   * Coverage
   * Duplications

---

## Step 12: (Optional) Enforce Quality Gate

### Add Quality Gate Stage

```groovy
stage('Quality Gate') {
    steps {
        timeout(time: 2, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
```

This will:

* Fail the pipeline if quality gate fails
* Enforce enterprise-level code quality standards

---

## Common Errors and Troubleshooting

| Issue                       | Cause                 | Fix                          |
| --------------------------- | --------------------- | ---------------------------- |
| Sonar authentication failed | Token missing         | Re-check Jenkins credentials |
| Project not visible         | Wrong project key     | Match key exactly            |
| Scanner not found           | Tool not configured   | Configure Sonar Scanner      |
| Port 9000 not reachable     | Firewall/Docker issue | Check container status       |

---

## Lab Outcome

You have successfully:

* Integrated SonarQube with Jenkins
* Automated static code analysis
* Enforced quality standards in CI pipeline
* Implemented industry-grade DevOps practices

---

