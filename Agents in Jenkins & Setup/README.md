
```
     ██╗███████╗███╗   ██╗██╗  ██╗██╗███╗   ██╗███████╗
     ██║██╔════╝████╗  ██║██║ ██╔╝██║████╗  ██║██╔════╝
     ██║█████╗  ██╔██╗ ██║█████╔╝ ██║██╔██╗ ██║███████╗
██   ██║██╔══╝  ██║╚██╗██║██╔═██╗ ██║██║╚██╗██║╚════██║
╚█████╔╝███████╗██║ ╚████║██║  ██╗██║██║ ╚████║███████║
 ╚════╝ ╚══════╝╚═╝  ╚═══╝╚═╝  ╚═╝╚═╝╚═╝  ╚═══╝╚══════╝
               A G E N T S  &  S E T U P
```

<div align="center">

![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![AWS EC2](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)

> **Scale your builds. Isolate your workloads. Run everywhere.**

</div>

---

## 📋 Table of Contents

- [What are Jenkins Agents?](#-what-are-jenkins-agents)
- [Types of Agents](#-types-of-agents)
  - [Static Agents](#-static-agents-permanent)
  - [Dynamic Agents](#-dynamic-agents-ephemeral)
- [Specific Agent Types](#-specific-agent-types)
- [Static vs Dynamic Comparison](#-static-vs-dynamic-comparison)
- [Setup: Static EC2 Agent](#-setup-static-ec2-agent)
- [Setup: Dynamic EC2 Agent](#-setup-dynamic-ec2-agent)
- [Best Practices](#-best-practices)
- [Summary](#-summary)

---

## 🤖 What are Jenkins Agents?

Jenkins agents (previously called "slaves") are **machines or environments that perform actual job execution**. The controller (master) delegates build tasks to agents, enabling Jenkins to:

```
  Jenkins Controller
        │
        │  Delegates build tasks
        │
   ┌────┴────────────────────────────────┐
   │              │                      │
   ▼              ▼                      ▼
Linux Agent   Windows Agent       Docker/K8s Agent
(Static)       (Static)            (Dynamic)
   │              │                      │
[Build A]     [Build B]             [Build C]
```

- **Scale horizontally** across many machines
- **Run jobs on different platforms** (Linux, Windows, macOS)
- **Isolate workloads** to prevent interference between builds
- **Maximize resource utilization** with parallel execution

---

## 🗂️ Types of Agents

All Jenkins agents fall under two core categories:

---

### 🖥️ Static Agents (Permanent)

Also called **dedicated agents** — configured manually and always available to the controller.

```
  Controller ◀──── Always Connected ────▶ Static Agent
                                           (Physical / VM)
                                           ┌─────────────┐
                                           │ Executor 1  │
                                           │ Executor 2  │
                                           └─────────────┘
```

**Best suited for:**
- Consistent environments requiring specific OS or pre-installed tools
- Long-running or heavy builds where startup time matters
- Dedicated hardware requirements (GPU, high-memory machines)

**Managed via:** `Manage Jenkins → Nodes`

---

### ☁️ Dynamic Agents (Ephemeral)

Also called **cloud agents** — automatically provisioned when a build is triggered and **destroyed after completion**.

```
  Build Triggered
        │
        ▼
  Controller requests new agent
        │
        ▼
  Agent spins up (Docker / K8s / Cloud VM)
        │
        ▼
  Build executes
        │
        ▼
  Agent destroyed ✅  ← no idle resource waste
```

**Best suited for:**
- Scalable CI/CD pipelines with variable load
- Cloud-native environments (Kubernetes, AWS, GCP, Azure)
- Reducing infrastructure costs — pay only when building

---

## 🔧 Specific Agent Types

### 🐳 Docker Agents

Agents run as Docker containers — isolated, reproducible, and fast.

**Workflow:**
```
  Jenkins pulls Docker image
        │
        ▼
  Starts container as agent
        │
        ▼
  Pipeline executes inside container
        │
        ▼
  Container removed after build ✅
```

**Plugins:** Docker Plugin, Docker Pipeline Plugin

---

### ☸️ Kubernetes Agents

Agents run as **Kubernetes pods**, dynamically created per pipeline build. Each stage can run in a separate pod.

**Features:**
- Fully ephemeral — pods spin up and down automatically
- Ideal for **scalable, cloud-native CI/CD**
- Native support for **parallel pipelines**
- No static infrastructure to manage

**Plugin:** Kubernetes Plugin

---

### 🌩️ Cloud Agents (AWS / Azure / GCP)

Dynamic agents provisioned as cloud VMs on demand.

**Workflow:**
```
  1. Controller requests cloud VM
         │
         ▼
  2. VM provisioned with required tools / AMI
         │
         ▼
  3. Build executes on VM
         │
         ▼
  4. VM terminated → cost saved ✅
```

**Plugins:** Amazon EC2 Plugin, Azure VM Agents Plugin, Google Cloud Plugin

---

### 🪟 Other Agent Types

| Type | Description |
|------|-------------|
| **Windows Agents** | Usually static, connected via JNLP or Windows Service |
| **Bare Metal Agents** | Dedicated servers for heavy builds or specific hardware |
| **External Agents** | Non-Jenkins-managed systems monitored externally |

---

## ⚖️ Static vs Dynamic Comparison

| Feature | Static Agent | Dynamic Agent |
|---------|-------------|---------------|
| Availability | Always online | Created on-demand |
| Provisioning | Manual | Automated via plugin |
| Scalability | Limited by fixed resources | Highly scalable |
| Resource Utilization | May sit idle | Efficient — destroyed after use |
| Setup Complexity | Moderate | Requires plugin & cloud config |
| Cost Model | Continuous running cost | Pay-per-use |
| Best Use Case | Dedicated builds, specific OS | CI/CD pipelines, parallel jobs |

---

## 🚀 Setup: Static EC2 Agent

A static EC2 agent is a **permanent Jenkins node** that stays online continuously.

---

### Prerequisites

Before you begin, ensure you have:

- Jenkins controller installed and accessible
- An AWS account with IAM permissions: `DescribeInstances`, `RunInstances`, `TerminateInstances`
- Plugins installed: **SSH Slaves Plugin**

---

### Step 1 — Launch an EC2 Instance

1. Go to **AWS Console → EC2 → Launch Instances**
2. Configure:
   - **AMI:** Ubuntu 22.04 LTS
   - **Instance type:** `t2.medium` (adjust based on build needs)
   - **Key pair:** Create or select an existing key pair
   - **Security group:** Allow **SSH (port 22)** from your Jenkins controller IP
3. Launch and note the **public/private IP**

---

### Step 2 — Prepare the EC2 Instance

SSH into the instance and install required tools:

```bash
ssh -i your-key.pem ubuntu@<ec2-public-ip>

sudo apt update
sudo apt install -y openjdk-17-jdk git maven docker.io
```

Verify Java is installed (Jenkins agent requires it):
```bash
java -version
```

---

### Step 3 — Add the Static Agent in Jenkins

1. Go to **Jenkins Dashboard → Manage Jenkins → Manage Nodes and Clouds → New Node**
2. Enter **Node Name:** `Static-EC2-Agent` → Select **Permanent Agent**
3. Configure:

| Field | Value |
|-------|-------|
| Remote root directory | `/home/ubuntu/jenkins` |
| Number of Executors | `2` |
| Labels | `linux ec2 static` |
| Launch Method | Launch agents via SSH |
| Host | EC2 public/private IP |
| Credentials | Add SSH private key credentials |
| Host Key Verification | Manually trusted key |
| Availability | Always |

4. **Save** and verify the agent comes **online** ✅

---

### Step 4 — Test the Static Agent

Create a freestyle job or pipeline, restrict it to label `linux ec2 static`, and trigger a build. Verify it runs on the correct node.

---

## 🔄 Setup: Dynamic EC2 Agent

Dynamic agents are **automatically created and terminated per build** using the Amazon EC2 Plugin.

---

### Step 1 — Install the Amazon EC2 Plugin

1. Go to **Manage Jenkins → Manage Plugins → Available**
2. Search for `Amazon EC2 Plugin`
3. Install and restart Jenkins

---

### Step 2 — Add AWS Credentials in Jenkins

1. Go to **Manage Jenkins → Credentials → System → Global Credentials → Add Credentials**
2. Configure:

| Field | Value |
|-------|-------|
| Kind | AWS Credentials |
| Access Key ID | Your AWS access key |
| Secret Access Key | Your AWS secret key |
| ID | `aws-ec2-credentials` |

---

### Step 3 — Configure EC2 Cloud in Jenkins

1. Go to **Manage Jenkins → Manage Nodes and Clouds → Configure Clouds → Add New Cloud → Amazon EC2**
2. Configure:

| Field | Value |
|-------|-------|
| Name | `EC2-Dynamic-Cloud` |
| Region | e.g., `us-east-1` |
| Credentials | `aws-ec2-credentials` |
| Jenkins URL | Your controller URL |
| Instance Cap | Max simultaneous instances (e.g., `10`) |

---

### Step 4 — Configure the EC2 Agent Template

Click **Add Template** and fill in:

| Field | Value |
|-------|-------|
| AMI ID | e.g., `ami-0abcdef1234567890` |
| Instance Type | `t2.medium` |
| Remote root directory | `/home/ubuntu/jenkins` |
| Labels | `linux ec2 dynamic` |
| Launch Method | SSH |
| SSH Username | `ubuntu` |
| Idle Termination Time | `5` minutes |
| Usage | Use this node as much as possible |

> 💡 **Tip:** Use a pre-baked AMI with Java, Git, Maven, and Docker already installed to reduce agent spin-up time.

---

### Step 5 — Configure Jobs to Use Dynamic Agents

**Freestyle Job:** Set *Restrict where this project can be run* → Label Expression: `linux ec2 dynamic`

**Declarative Pipeline:**

```groovy
pipeline {
    agent { label 'linux ec2 dynamic' }

    stages {
        stage('Build') {
            steps {
                sh 'echo "Running on dynamic EC2 agent"'
                sh 'mvn clean install'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
    }
}
```

---

### Step 6 — Test the Dynamic Agent

Trigger a build and watch Jenkins:

```
  Build triggered
       │
       ▼
  EC2 instance launched from template 🚀
       │
       ▼
  Pipeline stages execute on agent
       │
       ▼
  Idle for 5 mins → instance terminated 💀 → cost saved ✅
```

Verify in **AWS Console → EC2** that instances are being created and terminated automatically.

---

## ✅ Best Practices

| Recommendation | Reason |
|----------------|--------|
| Use **static agents** for heavy/specialized builds | Avoids EC2 spin-up latency |
| Use **dynamic agents** for CI/CD pipelines | Cost-efficient and scalable |
| Always **label agents** clearly | Ensures jobs run on the correct environment |
| Set **idle termination time** for dynamic agents | Prevents unnecessary AWS charges |
| Use **pre-baked AMIs** with required tools | Faster agent provisioning |
| **Limit instance cap** in EC2 cloud config | Avoids unexpected AWS bill spikes |
| Set **controller executors to 0** | Controller should orchestrate, not execute |

---

## 📌 Summary

| Feature | Static EC2 Agent | Dynamic EC2 Agent |
|---------|-----------------|-------------------|
| Availability | Always online | On-demand |
| Provisioning | Manual EC2 instance | Auto-launched from template |
| Cost | Continuous running cost | Pay-per-use, terminated after build |
| Scalability | Limited by fixed instances | Highly scalable |
| Best Use Case | Long-running, heavy builds | CI/CD pipelines, ephemeral jobs |
| Plugin Required | SSH Slaves Plugin | Amazon EC2 Plugin |

---

<div align="center">

*Part of the [Jenkins Zero to Hero](../README.md) course*

</div>
