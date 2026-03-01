<img src="https://github.com/bhuvan-raj/Jenkins-Zero-to-Hero/blob/main/assets/jenkins.png" alt="Banner" />

<div align="center">

![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Maven](https://img.shields.io/badge/Apache%20Maven-C71A36?style=for-the-badge&logo=apache-maven&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![GitHub Packages](https://img.shields.io/badge/GitHub%20Packages-181717?style=for-the-badge&logo=github&logoColor=white)

# Build & Deploy Maven Package to GitHub Packages

> **Compile. Package. Authenticate securely. Deploy to GitHub Packages — fully automated via Jenkins Pipeline.**

</div>

---

## 📋 Table of Contents

- [Lab Overview](#-lab-overview)
- [Prerequisites](#-prerequisites)
- [Step 1 — Create GitHub PAT Credential in Jenkins](#step-1--create-github-pat-credential-in-jenkins)
- [Step 2 — Add Maven settings.xml via Config File Management](#step-2--add-maven-settingsxml-via-config-file-management)
- [Step 3 — Set Up the Java Application](#step-3--set-up-the-java-application)
- [Step 4 — Configure pom.xml for GitHub Packages](#step-4--configure-pomxml-for-github-packages)
- [Step 5 — Create the Jenkins Pipeline Job](#step-5--create-the-jenkins-pipeline-job)
- [Step 6 — Write the Jenkinsfile](#step-6--write-the-jenkinsfile)
- [Step 7 — Run and Verify](#step-7--run-and-verify)
- [Lab Summary](#-lab-summary)

---

## 🎯 Lab Overview

In this lab, you will build a complete Jenkins Pipeline that:

1. Pulls a Java Maven project from GitHub
2. Builds and packages it into a JAR
3. Authenticates with GitHub using **secure Jenkins credentials** — no hardcoded secrets
4. Uses a managed `settings.xml` via **Config File Provider** for clean credential injection
5. Deploys the artifact to **GitHub Packages (Maven Registry)**

---

## ✅ Prerequisites

### Jenkins Setup

| Requirement | How to Configure |
|-------------|-----------------|
| Jenkins running | Accessible via browser |
| JDK configured | Manage Jenkins → Global Tool Configuration → JDK |
| Maven configured | Manage Jenkins → Global Tool Configuration → Maven |

### Jenkins Plugins Required

| Plugin | Purpose |
|--------|---------|
| **Pipeline** | Run Jenkinsfile-based pipelines |
| **Git** | Pull source code from GitHub |
| **Config File Provider** | Manage `settings.xml` securely |
| **GitHub Integration** | GitHub webhook and SCM integration |
| **Maven Integration** | Invoke Maven goals in Jenkins jobs |

Install any missing plugins via **Manage Jenkins → Manage Plugins → Available**.

### GitHub Requirements

- A GitHub repository containing your Maven project
- A **Personal Access Token (PAT)** with the following scopes:

| Scope | Purpose |
|-------|---------|
| `write:packages` | Push artifacts to GitHub Packages |
| `read:packages` | Read from GitHub Packages |
| `repo` | Access repository contents |

To create a PAT: **GitHub → Settings → Developer Settings → Personal Access Tokens → Generate new token**

---

## Step 1 — Create GitHub PAT Credential in Jenkins

Store your GitHub credentials securely in Jenkins so they can be referenced in pipelines without ever being hardcoded.

1. Go to **Manage Jenkins → Credentials → System → Global Credentials → Add Credentials**
2. Fill in:

| Field | Value |
|-------|-------|
| Kind | Username with password |
| Username | Your GitHub username |
| Password | Your GitHub PAT |
| ID | `github-packages-cred` |
| Description | GitHub Packages Deploy Credential |

3. Click **Save**

> 🔐 Jenkins encrypts credentials at rest. They are never exposed in logs — only referenced by ID inside pipelines.

---

## Step 2 — Add Maven settings.xml via Config File Management

Instead of storing a `settings.xml` in your repository (which risks exposing secrets), use Jenkins' **Config File Provider** to manage it centrally and inject it at build time.

1. Go to **Manage Jenkins → Managed Files**
2. Click **Add a new Config**
3. Select **Global Maven Settings File**
4. Configure:

| Field | Value |
|-------|-------|
| Name | `maven-github-settings` |
| ID | `maven-github-settings` |

5. Paste the following content:

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0">
  <servers>
    <server>
      <id>github</id>
      <username>${env.GH_USER}</username>
      <password>${env.GH_TOKEN}</password>
    </server>
  </servers>
</settings>
```

6. Click **Save**

The `${env.GH_USER}` and `${env.GH_TOKEN}` placeholders will be populated at runtime from environment variables set inside the Jenkinsfile — keeping credentials out of both the `settings.xml` and the repository.

---

## Step 3 — Set Up the Java Application

Your repository should have the following structure:

```
your-repo/
├── src/
│   └── main/
│       └── java/
│           └── com/example/
│               └── App.java
├── pom.xml
└── Jenkinsfile
```

**`src/main/java/com/example/App.java`**

```java
package com.example;

public class App {
    public static void main(String[] args) {
        System.out.println("Hello Jenkins CI/CD!");
    }
}
```

---

## Step 4 — Configure pom.xml for GitHub Packages

The `distributionManagement` section tells Maven where to deploy the artifact. The `<id>github</id>` must match the server `<id>` in your `settings.xml`.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>jenkins-demo</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <name>Jenkins GitHub Packages Demo</name>

    <distributionManagement>
        <repository>
            <id>github</id>
            <name>GitHub Packages</name>
            <url>https://maven.pkg.github.com/YOUR_USERNAME/YOUR_REPO</url>
        </repository>
    </distributionManagement>

</project>
```

> ⚠️ Replace `YOUR_USERNAME` and `YOUR_REPO` with your actual GitHub username and repository name before committing.

---

## Step 5 — Create the Jenkins Pipeline Job

1. Go to **Jenkins Dashboard → New Item**
2. Enter a name — e.g., `maven-deploy-pipeline`
3. Select **Pipeline** → Click **OK**
4. In the configuration page:
   - Under **Pipeline**, select **Pipeline script from SCM**
   - Set **SCM** to **Git**
   - Enter your repository URL
   - Add credentials if the repo is private
   - Set the branch (e.g., `*/main`)
   - Set **Script Path** to `Jenkinsfile`
5. Click **Save**

---

## Step 6 — Write the Jenkinsfile

Place this `Jenkinsfile` in the **root of your repository**:

```groovy
pipeline {
    agent any

    environment {
        GITHUB_CREDS = credentials('github-packages-cred')
        JAVA_HOME    = tool name: 'jdk11'
        MAVEN_HOME   = tool name: 'maven3'
        PATH         = "${JAVA_HOME}/bin:${PATH}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build & Deploy') {
            steps {
                configFileProvider([configFile(fileId: 'maven-github-settings', variable: 'MAVEN_SETTINGS')]) {
                    sh """
                        export GH_USER=${GITHUB_CREDS_USR}
                        export GH_TOKEN=${GITHUB_CREDS_PSW}

                        ${MAVEN_HOME}/bin/mvn -s $MAVEN_SETTINGS -B clean package
                        ${MAVEN_HOME}/bin/mvn -s $MAVEN_SETTINGS -B deploy
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build and deployment to GitHub Packages completed successfully."
        }
        failure {
            echo "❌ Pipeline failed. Check the console output for details."
        }
    }
}
```

**How the credential flow works:**

```
Jenkins Credentials Store
        │
        │  credentials('github-packages-cred')
        ▼
  GITHUB_CREDS_USR  ──▶  GH_USER  ──▶  settings.xml ${env.GH_USER}
  GITHUB_CREDS_PSW  ──▶  GH_TOKEN ──▶  settings.xml ${env.GH_TOKEN}
                                                │
                                                ▼
                                    Maven authenticates with GitHub Packages
```

> 💡 Jenkins automatically exposes `_USR` and `_PSW` suffixed variables when you use `credentials()` with a username/password credential type.

---

## Step 7 — Run and Verify

### Trigger the Build

1. Go to your pipeline job
2. Click **Build Now**

Jenkins will:
- Clone your GitHub repository
- Compile and package the JAR using Maven
- Inject credentials via `configFileProvider`
- Authenticate with GitHub Packages
- Deploy the artifact to the registry

### Check the Console Output

Click on **Build #1 → Console Output** and look for:

```
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  18.432 s
```

### Verify the Package on GitHub

Navigate to your GitHub repository:

```
GitHub → Your Repository → Packages (right sidebar)
```

You should see:

| Field | Value |
|-------|-------|
| Package Name | `jenkins-demo` |
| Version | `1.0.0` |
| Files | `jenkins-demo-1.0.0.jar`, `jenkins-demo-1.0.0.pom` |
| Registry | GitHub Maven Registry |

---

## 📌 Lab Summary

| Step | What Was Done |
|------|---------------|
| 🔐 Credentials | Stored GitHub PAT securely in Jenkins credentials store |
| 📄 settings.xml | Managed via Config File Provider — no secrets in repo |
| ☕ App Code | Created a simple Java `App.java` with Maven project structure |
| 📦 pom.xml | Configured `distributionManagement` pointing to GitHub Packages |
| ⚙️ Pipeline Job | Created Jenkins Pipeline job pointing to repo's `Jenkinsfile` |
| 🚀 Jenkinsfile | Wrote pipeline with `checkout`, `build`, `deploy` stages |
| ✅ Verify | Confirmed artifact visible in GitHub → Packages |

---

<div align="center">

*Part of the [Jenkins Zero to Hero](../README.md) course*

</div>
