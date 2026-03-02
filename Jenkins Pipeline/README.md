
<div align="center">

![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Groovy](https://img.shields.io/badge/Groovy-4298B8?style=for-the-badge&logo=apache-groovy&logoColor=white)
![Pipeline](https://img.shields.io/badge/Pipeline-as%20Code-brightgreen?style=for-the-badge)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

# Jenkins Pipeline

> **Define your entire CI/CD workflow as code. Version it. Review it. Automate it.**

</div>

---

## 📋 Table of Contents

- [What is a Jenkins Pipeline?](#-what-is-a-jenkins-pipeline)
- [Pipeline as Code — Jenkinsfile](#-pipeline-as-code--jenkinsfile)
- [Types of Pipelines](#-types-of-pipelines)
  - [Declarative Pipeline](#-declarative-pipeline-recommended)
  - [Scripted Pipeline](#-scripted-pipeline)
  - [Declarative vs Scripted Comparison](#-declarative-vs-scripted-comparison)
- [Pipeline Components](#-pipeline-components)
- [Advanced Concepts](#-advanced-concepts)
- [Common Pipeline Patterns](#-common-pipeline-patterns)
- [Best Practices](#-best-practices)
- [Summary](#-summary)

---

## 💡 What is a Jenkins Pipeline?

A **Jenkins Pipeline** is a suite of plugins that enables you to implement **Continuous Integration and Continuous Delivery (CI/CD)** workflows as code. Instead of configuring jobs through the UI, you write your entire build, test, and deploy process in a file — the **Jenkinsfile** — which lives in your source code repository.

**Why use Pipeline?**

| Benefit | Description |
|---------|-------------|
| 📄 Pipeline as Code | CI/CD logic is version-controlled alongside your application |
| 🔁 Repeatable | Every run follows the same defined process |
| 🔀 Parallel Execution | Run stages simultaneously to reduce build time |
| 🛡️ Reliable | Survive Jenkins restarts — durable pipeline execution |
| 👁️ Visual | Clear stage-by-stage visualization in the Jenkins UI |
| 🤝 Collaborative | Teams can review pipeline changes via pull requests |

---

## 📄 Pipeline as Code — Jenkinsfile

A **Jenkinsfile** is a text file placed in the root of your repository that defines the pipeline. It can be written in two syntaxes:

- **Declarative Pipeline** — structured, modern, recommended
- **Scripted Pipeline** — Groovy-based, flexible, for advanced use

```
  Your Repository
  ├── src/
  ├── pom.xml
  ├── Dockerfile
  └── Jenkinsfile          ← Pipeline definition lives here
```

When Jenkins picks up your repository, it reads the `Jenkinsfile` and executes the pipeline automatically. This means your CI/CD process is **reviewed, versioned, and rolled back** just like application code.

---

## 🔀 Types of Pipelines

### ✅ Declarative Pipeline (Recommended)

The **modern, preferred** syntax used by most teams and companies today. It is structured, readable, and validates your pipeline definition before execution.

**Characteristics:**
- Simple, opinionated syntax with clear structure
- Built-in error detection and validation
- Easier for teams to collaborate and maintain
- Best for the vast majority of CI/CD use cases

**Structure:**

```groovy
pipeline {
    agent any

    environment {
        APP_ENV = "production"
        VERSION = "1.0"
    }

    stages {
        stage('Build') {
            steps {
                echo "Building the application..."
                sh 'mvn clean install'
            }
        }
        stage('Test') {
            steps {
                echo "Running tests..."
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying to server..."
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check the logs."
        }
    }
}
```

---

### 🔧 Scripted Pipeline

The older, Groovy-based pipeline type that gives you **full programmatic control**. Used for complex, dynamic pipelines where declarative syntax is too limiting.

**Characteristics:**
- Full Groovy scripting flexibility
- No structural constraints
- Ideal for complex conditional logic
- Steeper learning curve

**Structure:**

```groovy
node {
    stage('Build') {
        echo "Building the application..."
        sh 'mvn clean install'
    }
    stage('Test') {
        echo "Running tests..."
        sh 'mvn test'
    }
    stage('Deploy') {
        echo "Deploying..."
    }
}
```

---

### ⚖️ Declarative vs Scripted Comparison

| Feature | Declarative | Scripted |
|---------|-------------|----------|
| Readability | ✅ High | ⚠️ Medium |
| Syntax Strictness | Structured & validated | Flexible — no enforced structure |
| Flexibility | Medium | Very High |
| Learning Curve | Easy | Hard (requires Groovy knowledge) |
| Error Handling | Built-in `post` blocks | Manual try/catch |
| Industry Adoption | ✅ Highly preferred | Used for advanced/special cases |
| Best For | Most CI/CD pipelines | Complex dynamic logic |

> 🏭 **Industry Verdict:** Most companies standardize on **Declarative Pipeline** for maintainability, readability, and easier onboarding of new team members.

---

## 🧩 Pipeline Components

A Declarative Pipeline is made up of several key sections, each with a specific responsibility:

### Agent

Defines **where** the pipeline (or a specific stage) will execute.

```groovy
agent any                          // Run on any available agent
agent none                         // No global agent — defined per stage
agent { label 'linux' }            // Run on agent with label 'linux'
agent { docker 'maven:3.9' }       // Run inside a Docker container
```

---

### Environment

Defines **environment variables** available throughout the pipeline or within a specific stage.

```groovy
environment {
    APP_ENV = "dev"
    VERSION  = "1.0"
    DB_URL   = "jdbc:mysql://localhost:3306/mydb"
}
```

---

### Stages & Steps

**Stages** are logical groupings of work (e.g., Build, Test, Deploy). **Steps** are the actual commands that execute inside a stage.

```groovy
stages {
    stage('Build') {
        steps {
            sh 'mvn clean install'
        }
    }
    stage('Test') {
        steps {
            sh 'mvn test'
        }
    }
}
```

---

### Post Section

Defines actions to take **after** the pipeline or a stage completes, based on the result.

```groovy
post {
    always  { echo "This always runs" }
    success { echo "Build succeeded ✅" }
    failure { echo "Build failed ❌ — notify team" }
    cleanup { cleanWs() }
}
```

---

## 🚀 Advanced Concepts

### Parallel Stages

Run multiple stages **simultaneously** to cut total pipeline time.

```groovy
stage('Run Tests in Parallel') {
    parallel {
        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Integration Tests') {
            steps {
                sh './run_integration.sh'
            }
        }
        stage('Security Scan') {
            steps {
                sh './security_scan.sh'
            }
        }
    }
}
```

---

### Input Step — Manual Approval Gate

Pause the pipeline and **wait for a human to approve** before proceeding (e.g., before deploying to production).

```groovy
stage('Approval') {
    steps {
        input message: "Deploy to production?", ok: "Yes, Deploy"
    }
}
```

---

### Docker Agent in Pipeline

Run your pipeline inside a Docker container — giving you a clean, isolated, reproducible build environment every time.

```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.9-eclipse-temurin-17'
            args  '-v /tmp:/tmp'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}
```

---

### Shared Libraries

Reuse common pipeline logic across multiple repositories by storing it in a **centralized shared library**.

```groovy
@Library('my-shared-lib') _

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                mySharedBuildStep()   // function from shared library
            }
        }
    }
}
```

Shared libraries are ideal for large organizations running dozens of pipelines that share common steps like notifications, deployments, or artifact publishing.

---

### Credentials Management

Never hardcode secrets. Use Jenkins' built-in **credentials store** to securely inject secrets into your pipeline.

```groovy
withCredentials([
    usernamePassword(
        credentialsId: 'docker-hub-creds',
        usernameVariable: 'DOCKER_USER',
        passwordVariable: 'DOCKER_PASS'
    )
]) {
    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
}
```

Supported credential types: username/password, SSH keys, secret text, AWS keys, certificates.

---

### Stage-Level Agents

When using `agent none` globally, you can target **different agents per stage** — useful for cross-platform builds.

```groovy
pipeline {
    agent none

    stages {
        stage('Build on Linux') {
            agent { label 'linux' }
            steps { sh 'mvn clean install' }
        }
        stage('Build on Windows') {
            agent { label 'windows' }
            steps { bat 'mvn clean install' }
        }
    }
}
```

---

## 🔁 Common Pipeline Patterns

**Build → Test → Package → Deploy** — the standard CI/CD flow for most applications.

**Build Matrix** — run the same pipeline across multiple OS versions or JDK versions simultaneously using parallel stages or Multiconfiguration jobs.

**Rolling Deployments** — deploy to a subset of servers at a time, commonly used with Kubernetes or Ansible to minimize downtime.

**Canary Deployments** — route a small percentage of traffic to the new version first, monitor, then roll out fully if healthy.

**Manual Gated Deployments** — use the `input` step to require human sign-off before promoting to production or staging.

---

## ✅ Best Practices

| Practice | Why It Matters |
|----------|----------------|
| Always use **Declarative Pipeline** unless complex Groovy logic is needed | Better readability and maintainability |
| Store `Jenkinsfile` in the **repository root** | Keeps CI/CD co-located with application code |
| Use **`agent none`** globally and define agents at stage level | Gives fine-grained control over where each stage runs |
| Split large pipelines using **Shared Libraries** | Avoids duplication across many repos |
| Use proper **`post`** blocks for cleanup and notifications | Ensures cleanup even on failure |
| Use **`withCredentials`** for all secrets | Prevents secrets from leaking into logs |
| Set **`timeout`** on long-running stages | Prevents hung builds from blocking executors |
| Use **`retry`** cautiously | Good for flaky tests — but masks real issues if overused |
| Use meaningful **stage names** | Makes the pipeline visualization immediately understandable |

---

## 📌 Summary

Jenkins Pipeline is the backbone of modern CI/CD. Here's the key takeaway:

| Topic | Key Point |
|-------|-----------|
| **Jenkinsfile** | Defines your pipeline as code, stored in your repo |
| **Declarative Pipeline** | Structured, readable, validated — preferred by industry |
| **Scripted Pipeline** | Full Groovy flexibility — for advanced dynamic logic |
| **Parallel Stages** | Run multiple stages simultaneously to save time |
| **Input Step** | Add manual approval gates before sensitive deployments |
| **Docker Agent** | Clean, reproducible build environments per run |
| **Shared Libraries** | Centralize and reuse pipeline logic across repos |
| **Credentials Store** | Secure secret injection — never hardcode secrets |

---

<div align="center">

*Part of the [Jenkins Zero to Hero](../README.md) course*

</div>
