<img src="https://github.com/bhuvan-raj/Jenkins-Zero-to-Hero/blob/main/assets/jenkins.png" alt="Banner" />

<div align="center">

![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Pipeline](https://img.shields.io/badge/Pipeline-as%20Code-blue?style=for-the-badge)
![Maven](https://img.shields.io/badge/Apache%20Maven-C71A36?style=for-the-badge&logo=apache-maven&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)

# Jenkins Job Types

> **Every job type explained — when to use it, how to set it up, and what it looks like in practice.**

</div>

---

## 📋 Table of Contents

- [What is a Jenkins Job?](#-what-is-a-jenkins-job)
- [1. Freestyle Project](#1️⃣-freestyle-project)
- [2. Maven Project](#2️⃣-maven-project)
- [3. Pipeline Job](#3️⃣-pipeline-job)
- [4. Multibranch Pipeline](#4️⃣-multibranch-pipeline)
- [5. GitHub Organization Job](#5️⃣-github-organization-job)
- [6. Folder Job](#6️⃣-folder-job)
- [7. External Job](#7️⃣-external-job)
- [8. Matrix / Multiconfiguration Job](#8️⃣-matrix--multiconfiguration-job)
- [Summary Table](#-summary-table)

---

## 💡 What is a Jenkins Job?

A **Jenkins job** (also called a project) is a task or automated workflow that Jenkins executes. Jobs can build code, run tests, deploy applications, run scripts, or monitor external processes.

Jenkins supports multiple job types — each designed for a specific automation scenario, from simple shell scripts to complex multi-branch, multi-environment CI/CD pipelines.

---

## 1️⃣ Freestyle Project

The **Freestyle Project** is the most basic and flexible Jenkins job type. It offers a point-and-click UI for configuration with no code required.

**Best for:** Simple builds, running scripts, basic SCM integration, one-off automation tasks.

### How to Create

1. Go to **Jenkins Dashboard → New Item**
2. Enter a job name → Select **Freestyle project** → Click **OK**
3. Configure each tab:

| Tab | What to Configure |
|-----|------------------|
| **General** | Description, discard old builds, restrict where job runs |
| **Source Code Management** | Git/SVN repository URL, credentials, branch |
| **Build Triggers** | Poll SCM, GitHub webhook, cron schedule |
| **Build Environment** | Clean workspace, environment variables |
| **Build Steps** | Execute shell / batch, invoke Maven/Ant/Gradle |
| **Post-build Actions** | Archive artifacts, send email, trigger downstream jobs |

4. Click **Save → Build Now**

---

## 2️⃣ Maven Project

The **Maven Project** is specifically designed for Java applications that use Apache Maven as the build tool. Jenkins understands the Maven lifecycle natively, making configuration simpler than freestyle.

**Best for:** Java applications, dependency management, Maven goals like `compile`, `test`, `package`.

### How to Create

1. Install the **Maven Integration Plugin** via *Manage Plugins* (if not already installed)
2. Go to **New Item → Enter job name → Maven project → OK**
3. Configure:

| Field | Details |
|-------|---------|
| Source Code Management | Git/SVN repository, credentials, branch |
| Build Triggers | Poll SCM, GitHub webhook, timer |
| Root POM | Path to `pom.xml` (default: root of repo) |
| Goals | e.g., `clean install`, `clean test`, `clean package` |
| Post-build Actions | Archive artifacts, trigger downstream jobs |

4. Click **Save → Build Now**

---

## 3️⃣ Pipeline Job

The **Pipeline Job** is the modern, preferred way to define CI/CD in Jenkins. Pipelines are written as code in a `Jenkinsfile` and stored alongside the application in version control.

**Best for:** Complex multi-stage CI/CD workflows, parallel builds, version-controlled pipeline definitions.

### How to Create

1. Go to **New Item → Pipeline → OK**
2. Configure:
   - **General:** Description, discard old builds
   - **Build Triggers:** SCM webhook, cron schedule
   - **Pipeline Definition:** Choose one of:
     - **Pipeline script** — write Groovy directly in Jenkins UI
     - **Pipeline script from SCM** — load `Jenkinsfile` from your repository (recommended)

3. Example **Declarative Pipeline:**

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                sh 'mvn clean install'
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}
```

4. Click **Save → Build Now**

> 💡 **Tip:** Always prefer **Pipeline script from SCM** so your pipeline definition lives in Git alongside your code — giving you version history, code reviews, and rollback capability.

---

## 4️⃣ Multibranch Pipeline

The **Multibranch Pipeline** automatically discovers branches in a repository and creates a separate pipeline job for each branch that contains a `Jenkinsfile`. When a new branch is pushed, Jenkins automatically sets up the pipeline.

**Best for:** Git workflows with feature branches, pull request builds, branch-specific pipelines.

### How it Works

```
  GitHub Repository
  ├── main          ──▶  Pipeline: my-app/main
  ├── feature/login ──▶  Pipeline: my-app/feature-login
  ├── feature/api   ──▶  Pipeline: my-app/feature-api
  └── hotfix/bug-42 ──▶  Pipeline: my-app/hotfix-bug-42
```

### How to Create

1. Install the **Multibranch Pipeline Plugin**
2. Go to **New Item → Multibranch Pipeline → OK**
3. Configure:

| Field | Details |
|-------|---------|
| Branch Sources | Git, GitHub, or Bitbucket — provide repo URL and credentials |
| Discover Branches | Enable automatic branch detection |
| Discover PRs | Enable to build pull requests automatically |
| Build Configuration | Set path to `Jenkinsfile` (default: root of repo) |
| Scan Triggers | Set scan interval to detect new branches automatically |

4. Click **Save** — Jenkins scans the repository and creates pipelines per branch automatically.

---

## 5️⃣ GitHub Organization Job

The **GitHub Organization Job** scans an entire GitHub organization and automatically discovers all repositories. For any repository containing a `Jenkinsfile`, Jenkins creates and manages a multibranch pipeline automatically.

**Best for:** Large GitHub organizations, managing dozens of repositories automatically, enforcing CI/CD across all projects.

### How to Create

1. Go to **New Item → GitHub Organization → OK**
2. Configure:

| Field | Details |
|-------|---------|
| Owner | GitHub organization name |
| Credentials | GitHub personal access token with `repo` scope |
| Repository Filter | Optionally filter repos by name pattern |
| Build Configuration | Path to `Jenkinsfile` |
| Scan Triggers | How frequently Jenkins re-scans for new repos |

3. Click **Save** — Jenkins scans the org and creates pipelines for every repository that has a `Jenkinsfile`.

> ℹ️ New repositories added to the org are picked up automatically on the next scan — no manual job creation needed.

---

## 6️⃣ Folder Job

**Folders** are containers used to organize related jobs. They support folder-level credentials, permissions, and views — making large Jenkins instances much easier to manage.

**Best for:** Organizing jobs by team, project, or environment. Applying scoped credentials and access control.

### How to Create

1. Go to **New Item → Folder → OK**
2. Enter a folder name and description
3. Create jobs inside the folder — they'll be scoped to it
4. Configure folder-level credentials or permissions as needed

> 💡 **Tip:** Combine folders with role-based access control (RBAC) so each team only sees and manages their own jobs.

---

## 7️⃣ External Job

The **External Job** type allows Jenkins to **monitor builds that run outside of Jenkins** — such as cron jobs, legacy scripts, or builds on external systems.

**Best for:** Legacy build processes, external cron jobs, scripts running on non-Jenkins infrastructure.

### How to Create

1. Go to **New Item → External Job → OK**
2. Configure:
   - Description
   - Location or hostname of the external job
   - Build triggers (poll external status)
3. Jenkins monitors the execution and captures results for visibility in the dashboard.

---

## 8️⃣ Matrix / Multiconfiguration Job

The **Matrix Project** (also called Multiconfiguration) runs the **same job across multiple environments or configurations simultaneously**. You define axes (e.g., OS, JDK version, Node label) and Jenkins generates all combinations, running them in parallel.

**Best for:** Cross-platform testing, validating builds across multiple JDK/OS versions, matrix testing strategies.

### Example Matrix

| Axis | Values |
|------|--------|
| OS | `ubuntu`, `windows` |
| JDK | `java11`, `java17` |

Jenkins will run: `ubuntu × java11`, `ubuntu × java17`, `windows × java11`, `windows × java17` — **4 parallel builds automatically**.

### How to Create

1. Go to **New Item → Multiconfiguration project → OK**
2. Configure:

| Field | Details |
|-------|---------|
| Axes | Define variables — OS type, JDK version, Node label |
| Build Steps | Steps are the same across all configurations |
| Execution Strategy | Run combinations sequentially or in parallel |
| Post-build Actions | Archive results, send notifications |

3. Click **Save → Build Now** — Jenkins spawns parallel builds for each configuration combination.

---

## 📊 Summary Table

| Job Type | Best For | Requires Plugin |
|----------|----------|-----------------|
| **Freestyle Project** | Simple builds, shell scripts, basic SCM integration | No |
| **Maven Project** | Java projects with Maven build automation | Maven Integration Plugin |
| **Pipeline** | Code-defined CI/CD, complex multi-stage workflows | Pipeline Plugin (built-in) |
| **Multibranch Pipeline** | Auto-pipeline per branch, PR builds | Multibranch Pipeline Plugin |
| **GitHub Organization** | Auto-manage all repos in a GitHub org | GitHub Branch Source Plugin |
| **Folder** | Organizing and grouping related jobs | No |
| **External Job** | Monitoring builds that run outside Jenkins | No |
| **Matrix / Multiconfiguration** | Parallel builds across multiple OS/JDK environments | No |

---

<div align="center">

*Part of the [Jenkins Zero to Hero](../README.md) course*

</div>
