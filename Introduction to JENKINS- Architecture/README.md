<img src="https://github.com/bhuvan-raj/Jenkins-Zero-to-Hero/blob/main/assets/jenkins.png" alt="Banner" />


<div align="center">

![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Distributed](https://img.shields.io/badge/Distributed-Builds-blue?style=for-the-badge)
![Plugins](https://img.shields.io/badge/Plugins-1800%2B-brightgreen?style=for-the-badge)
![Open Source](https://img.shields.io/badge/Open%20Source-❤️-red?style=for-the-badge)

> **The brain, the agents, and everything in between.**

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [High-Level Architecture](#-high-level-architecture)
  - [Jenkins Controller (Master)](#-jenkins-controller-master)
  - [Jenkins Agent (Node)](#-jenkins-agent-node)
- [Internal Components](#-jenkins-internal-components)
  - [Controller Components](#-controller-components)
  - [Executors — Core Concept](#-executors--core-concept)
  - [Agent & Node Components](#-agent--node-components)
- [Plugins](#-plugins--the-heart-of-jenkins)
- [Build Pipeline Architecture](#-build-pipeline-architecture)
- [Distributed Build Architecture](#-distributed-build-architecture)
- [Storage Architecture](#-storage-architecture)
- [Security Architecture](#-security-architecture)
- [Lifecycle of a Jenkins Build](#-lifecycle-of-a-jenkins-build)
- [Quick Summary](#-quick-summary)

---

## 🔍 Overview

Jenkins is an **open-source automation server** primarily used for Continuous Integration (CI) and Continuous Delivery (CD). Its architecture is designed to be:

- **Modular** — built around a powerful plugin system
- **Distributed** — scales horizontally using controller-agent architecture
- **Extensible** — 1,800+ plugins to integrate with virtually any tool

Jenkins automates the **build, test, and deployment** of applications across diverse environments.

---

## 🏗️ High-Level Architecture

```
                        ┌─────────────────────────────┐
                        │      JENKINS CONTROLLER      │
                        │  ┌────────┐  ┌───────────┐  │
         Developers ───▶│  │ Web UI │  │ REST API  │  │
         (Git Push)     │  └────────┘  └───────────┘  │
                        │  ┌──────────────────────┐   │
         Webhooks  ───▶ │  │    Build Queue       │   │
                        │  └──────────────────────┘   │
                        │  ┌──────────────────────┐   │
                        │  │   Job Orchestration  │   │
                        │  └──────────────────────┘   │
                        └──────────┬──────────────────┘
                                   │
                    Dispatches builds to agents
                                   │
           ┌───────────────────────┼───────────────────────┐
           │                       │                       │
  ┌────────▼────────┐   ┌──────────▼────────┐   ┌─────────▼────────┐
  │   AGENT (Linux) │   │  AGENT (Windows)  │   │  AGENT (Docker)  │
  │  Executor 1 ✅  │   │  Executor 1 ✅    │   │  Executor 1 ✅   │
  │  Executor 2 🔄  │   │  Executor 2 🔄    │   │  Executor 2 ⏳   │
  └─────────────────┘   └───────────────────┘   └──────────────────┘
```

---

### 🧠 Jenkins Controller (Master)

The **Controller** is the central brain of Jenkins. It is responsible for:

| Responsibility | Details |
|----------------|---------|
| 🖥️ Web UI & REST API | Serves dashboards, logs, real-time console output |
| ⚙️ Job Configuration | Stores and manages all job/pipeline definitions |
| 📋 Queue Management | Holds pending builds until an executor is available |
| 🚦 Build Dispatching | Assigns jobs to appropriate agents based on labels |
| 🔌 Plugin Management | Installs, updates, and manages plugins |
| 🔐 Credentials Store | Securely stores SSH keys, tokens, secrets |
| 📁 Log & History Storage | Retains build history and output logs |

> ℹ️ In modern Jenkins, "Master" has been officially renamed to **Controller**.

---

### 🤖 Jenkins Agent (Node)

A Jenkins **Agent** is any machine — physical server, VM, Docker container, or Kubernetes pod — that **executes the actual build jobs**.

Agents are used to:
- Offload workloads from the controller
- Scale horizontally across many machines
- Run platform-specific builds (Linux / Windows / macOS)
- Run multiple jobs in parallel

**Agent Communication Protocols:**

| Protocol | Use Case |
|----------|----------|
| SSH | Standard Linux/Unix agents |
| JNLP / WebSocket | Cloud, Windows, firewalled environments |
| Docker Plugin | Temporary containerized agents |
| Kubernetes Plugin | Dynamic, scalable pod-based agents |

---

## 🔧 Jenkins Internal Components

### 🖥️ Controller Components

#### Web UI & CLI Layer
The controller exposes a **web dashboard** showing job views, build logs, and real-time console output. The **Jenkins CLI** allows admins to perform operations (create jobs, trigger builds, manage nodes) via command line.

#### Job / Project Configurations
All job configurations are stored as **XML files** in the Jenkins home directory.

```
JENKINS_HOME/
├── jobs/
│   ├── my-pipeline/
│   │   └── config.xml
│   └── deploy-job/
│       └── config.xml
├── plugins/
├── credentials.xml
└── config.xml
```

Supported job types: Freestyle, Declarative Pipeline, Scripted Pipeline, Multibranch Pipeline, GitHub Organization.

#### Build Queue
When a build is triggered, it enters the **queue**. The controller evaluates:
- Whether agents have available executors
- Node label requirements
- Resource availability and job restrictions

Only when all conditions are satisfied is the job dispatched to an agent.

---

### ⚡ Executors — Core Concept

An **executor** is a slot in which Jenkins runs a single build at a time.

```
  AGENT NODE (4 Executors)
  ┌──────────────────────────────────┐
  │  Executor 1 │ Running: job-A ✅  │
  │  Executor 2 │ Running: job-B ✅  │
  │  Executor 3 │ Running: job-C ✅  │
  │  Executor 4 │ IDLE ⏳            │
  └──────────────────────────────────┘
```

- Each executor runs **one job at a time**
- More executors = more **parallel builds**
- Executors consume the **CPU and RAM** of the host machine

**Executor Ownership:**

| Location | Default Executors |
|----------|------------------|
| Controller | 2 (should be set to 0 in production) |
| Agent | Configurable based on machine resources |

**Best Practices:**

> ✅ Set controller executors to **0** — the controller should only orchestrate, never execute builds.  
> ✅ Size agent executors based on available **CPU and RAM**.  
> ❌ Avoid over-allocating executors — this causes resource exhaustion and build failures.

---

### 🖧 Agent & Node Components

#### Node Labels
Labels allow you to target specific agents for specific jobs.

```groovy
pipeline {
    agent { label 'linux && docker' }
    ...
}
```

Common label examples: `linux`, `windows`, `docker`, `java17`, `gpu`, `high-memory`

#### Workspace
Each build gets its own **workspace directory** on the agent:

```
/var/jenkins/workspace/my-pipeline/
├── src/
├── target/
├── Jenkinsfile
└── pom.xml
```

The workspace contains the source code checkout, build artifacts, and temporary files. It can be cleaned up after builds based on configuration.

#### Tools Configuration
Jenkins manages tool versions per agent, including JDK, Maven, Gradle, NodeJS, and Python. These are configured globally and used during build execution.

---

## 🔌 Plugins — The Heart of Jenkins

Jenkins has **1,800+ plugins** that extend almost every aspect of its functionality.

```
  Jenkins Core
       │
       ├── SCM Plugins ──────── Git, GitHub, Bitbucket, GitLab
       ├── Build Tools ──────── Maven, Gradle, Docker, NodeJS
       ├── Credentials ──────── Credentials Binding, HashiCorp Vault
       ├── Auth & Access ────── LDAP, SSO, Matrix Authorization, RBAC
       ├── Cloud & Agents ───── Kubernetes, EC2, Docker agents
       └── Visualization ────── Blue Ocean, Pipeline Graph View
```

> ⚠️ Plugins rely on one another and form **dependency trees**. A broken or incompatible plugin can affect the entire Jenkins instance — always test plugin updates in a non-production environment first.

---

## 🔄 Build Pipeline Architecture

### Jenkinsfile (Pipeline as Code)

Pipelines are defined in a **Jenkinsfile** committed to the repository alongside application code.

```groovy
pipeline {
    agent { label 'linux' }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                sh './deploy.sh'
            }
        }
    }
}
```

| Pipeline Type | Description |
|---------------|-------------|
| **Declarative** | Structured, opinionated syntax — recommended for most use cases |
| **Scripted** | Full Groovy flexibility — for complex, dynamic pipelines |

Pipelines support **parallel stages**, **conditional flows**, and **shared libraries**.

---

## 🌐 Distributed Build Architecture

### When to Use Distributed Builds

- Heavy or long-running builds
- Multi-platform testing (Linux + Windows + macOS)
- Build isolation requirements
- Large teams with many concurrent jobs

### Controller–Agent Communication

```
  Controller                           Agent
     │                                   │
     │── SSH / JNLP / WebSocket ────────▶│
     │── Job instructions ───────────────▶│
     │◀─ Build logs & status ────────────│
     │◀─ Artifacts ──────────────────────│
```

### Scalability

Jenkins scales **horizontally** by:
- Adding static agent nodes
- Dynamically provisioning agents via **Kubernetes**, **AWS EC2**, or **Docker** plugins — agents spin up when needed and tear down after the job completes

---

## 💾 Storage Architecture

```
JENKINS_HOME/
├── jobs/                  # Job configs, build history, logs
│   └── <job-name>/
│       ├── config.xml
│       └── builds/
│           └── 42/
│               ├── log
│               └── archive/
├── plugins/               # Installed plugin JARs and data
├── credentials.xml        # Encrypted credentials store
├── nodes/                 # Agent node configurations
└── config.xml             # Global Jenkins configuration
```

**Artifact Storage Options:**

| Option | Use Case |
|--------|----------|
| Local disk | Simple setups, small teams |
| Nexus / Artifactory | Enterprise artifact management |
| GitHub Packages | GitHub-integrated projects |
| S3 / Cloud storage | Scalable cloud environments |

Logs can be shipped externally to **CloudWatch**, **Elasticsearch**, or **Splunk** via plugins.

---

## 🔐 Security Architecture

### Authentication

| Method | Description |
|--------|-------------|
| Jenkins Internal Users | Default username/password database |
| LDAP / Active Directory | Enterprise directory integration |
| SSO (SAML / OIDC) | Single sign-on via identity providers |
| GitHub OAuth | Login with GitHub account |

### Authorization

| Model | Description |
|-------|-------------|
| Full Access | All users have full control (not recommended for production) |
| Project-Based Matrix | Fine-grained per-project permissions |
| Role-Based Access Control | Assign roles to users/groups via RBAC plugin |

### Credentials Store

Sensitive data is **encrypted with a master key** and stored in Jenkins. Accessed in pipelines via the credentials binding plugin.

```groovy
withCredentials([usernamePassword(
    credentialsId: 'docker-hub',
    usernameVariable: 'USER',
    passwordVariable: 'PASS'
)]) {
    sh 'docker login -u $USER -p $PASS'
}
```

Supported credential types: SSH keys, AWS keys, API tokens, secret text, certificates.

---

## 🔁 Lifecycle of a Jenkins Build

```
  1. Developer commits code
          │
          ▼
  2. SCM Webhook triggers Jenkins
          │
          ▼
  3. Build enters the Queue
          │
          ▼
  4. Controller finds a matching agent + free executor
          │
          ▼
  5. Workspace is prepared on the agent
          │
          ▼
  6. Pipeline stages & steps execute
          │
          ▼
  7. Logs stream to UI in real-time
          │
          ▼
  8. Artifacts archived
          │
          ▼
  9. Post-build actions run (email, Slack, deploy, notify)
          │
          ▼
  10. Executor freed → ready for next job
```

---

## 📌 Quick Summary

| Component | Role |
|-----------|------|
| **Controller** | Orchestrates jobs, stores configs, manages the build queue |
| **Agent** | Executes builds on dedicated machines |
| **Executor** | A slot that runs one job at a time; determines parallelism |
| **Plugins** | Extend Jenkins functionality — 1,800+ available |
| **Pipeline / Jenkinsfile** | Defines CI/CD automation as code |
| **Distributed Architecture** | Multiple agents enable horizontal scaling |
| **Security** | Authentication + Authorization + Encrypted credential store |

---

<div align="center">

*Part of the [Jenkins Zero to Hero](../README.md) course*

</div>
