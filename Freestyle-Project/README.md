
<div align="center">

![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Git](https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)
![Shell](https://img.shields.io/badge/Shell-Script-4EAA25?style=for-the-badge&logo=gnu-bash&logoColor=white)

# Jenkins Freestyle Project

> **The simplest way to automate a task in Jenkins — no code required.**

</div>

---

## 📋 Table of Contents

- [What is a Freestyle Project?](#-what-is-a-freestyle-project)
- [When to Use It](#-when-to-use-it)
- [Freestyle vs Pipeline](#-freestyle-vs-pipeline)
- [Lab: Create Your First Freestyle Job](#-lab-create-your-first-freestyle-job)
  - [Step 1 — Create the Job](#step-1--create-the-job)
  - [Step 2 — General Settings](#step-2--general-settings)
  - [Step 3 — Connect a GitHub Repository](#step-3--connect-a-github-repository)
  - [Step 4 — Set Build Triggers](#step-4--set-build-triggers)
  - [Step 5 — Configure Build Environment](#step-5--configure-build-environment)
  - [Step 6 — Add Build Steps](#step-6--add-build-steps)
  - [Step 7 — Add Post-Build Actions](#step-7--add-post-build-actions)
  - [Step 8 — Save and Run](#step-8--save-and-run)
- [Understanding the Configuration Tabs](#-understanding-the-configuration-tabs)
- [Common Build Step Examples](#-common-build-step-examples)
- [Troubleshooting](#-troubleshooting)
- [Summary](#-summary)

---

## 💡 What is a Freestyle Project?

A **Freestyle Project** is Jenkins' most fundamental job type. It provides a **graphical, point-and-click interface** to configure automation tasks without writing any pipeline code.

You configure everything through the Jenkins UI — source code, triggers, build steps, and post-build actions — and Jenkins takes care of the rest.

Think of it as a **building block**: easy to set up, flexible enough for many use cases, and a great starting point for understanding how Jenkins works before moving to Pipeline-as-Code.

---

## ✅ When to Use It

| Good Fit | Not Ideal For |
|----------|--------------|
| Simple build or shell script automation | Complex multi-stage CI/CD workflows |
| Running a single Maven / Gradle / Ant goal | Pipelines that need version control |
| Quick integration with SCM and webhooks | Parallel execution across agents |
| Learning and experimenting with Jenkins | Large team collaboration on build logic |
| One-off or scheduled automation tasks | Reusable, shared pipeline definitions |

---

## ⚖️ Freestyle vs Pipeline

| Feature | Freestyle Project | Pipeline (Jenkinsfile) |
|---------|------------------|----------------------|
| Configuration | UI-based (click & fill) | Code-based (Groovy) |
| Version Control | ❌ Not versioned | ✅ Stored in Git |
| Complex Workflows | Limited | Full support |
| Parallel Stages | ❌ Not supported | ✅ Supported |
| Reusability | Low | High (Shared Libraries) |
| Visibility | Basic | Visual stage view |
| Best For | Simple tasks, learning | Production CI/CD |

> 💡 **Recommendation:** Use Freestyle for learning and simple automation. Migrate to Pipeline jobs for anything going to production.

---

## 🧪 Lab: Create Your First Freestyle Job

In this lab, you will create a Freestyle job that:
- Pulls source code from a GitHub repository
- Executes a shell script build step
- Archives the output artifact
- Sends a notification after the build

---

### Step 1 — Create the Job

1. From the **Jenkins Dashboard**, click **New Item**
2. Enter a job name — e.g., `my-first-freestyle-job`
3. Select **Freestyle project**
4. Click **OK**

You will land on the job configuration page with several tabs.

---

### Step 2 — General Settings

At the top of the configuration page, fill in the **General** section:

| Field | Recommended Value |
|-------|------------------|
| Description | A short description of what this job does |
| Discard old builds | ✅ Enable — set **Max # of builds to keep** to `5` |
| Restrict where this project can be run | Optional — set a node label if targeting a specific agent |

Enabling **Discard old builds** keeps your Jenkins storage clean by automatically removing old build logs and artifacts.

---

### Step 3 — Connect a GitHub Repository

1. Scroll to the **Source Code Management** section
2. Select **Git**
3. Fill in:

| Field | Value |
|-------|-------|
| Repository URL | `https://github.com/<your-username>/<your-repo>.git` |
| Credentials | Add credentials if private repo (username + PAT) |
| Branch Specifier | `*/main` |

Jenkins will clone this repository into the job's workspace every time the build runs.

> 💡 For **public repositories**, no credentials are needed. For **private repositories**, add a GitHub Personal Access Token (PAT) as a **Username with password** credential via **Manage Jenkins → Credentials**.

---

### Step 4 — Set Build Triggers

Build triggers define **what causes this job to run**. Scroll to the **Build Triggers** section and choose one or more:

| Trigger | When to Use |
|---------|-------------|
| **Build periodically** | Schedule builds using cron syntax — e.g., `H 8 * * 1-5` runs every weekday at 8 AM |
| **Poll SCM** | Jenkins periodically checks the repo for new commits — e.g., `H/5 * * * *` checks every 5 mins |
| **GitHub hook trigger** | Recommended — GitHub sends a webhook to Jenkins on every push, triggering an instant build |
| **Build after other projects** | Trigger this job after another Jenkins job completes |

**Recommended cron examples:**

```
H/5 * * * *       # Every 5 minutes
H 8 * * 1-5       # Weekdays at 8 AM
0 0 * * *         # Every day at midnight
H H * * 0         # Once a week on Sunday
```

---

### Step 5 — Configure Build Environment

Scroll to **Build Environment** and enable any of the following as needed:

| Option | Purpose |
|--------|---------|
| **Delete workspace before build starts** | Ensures a clean build every time — no leftover files from previous runs |
| **Add timestamps to Console Output** | Adds a timestamp to every line of console log — useful for debugging |
| **Use secret text(s) or file(s)** | Inject credentials as environment variables into the build |

---

### Step 6 — Add Build Steps

This is where the actual work happens. Click **Add build step** and choose the appropriate option.

#### Option A — Execute Shell (Linux/macOS agents)

```bash
#!/bin/bash
echo "=============================="
echo "Starting build..."
echo "=============================="

# Print environment info
echo "Branch: $GIT_BRANCH"
echo "Build Number: $BUILD_NUMBER"
echo "Workspace: $WORKSPACE"

# Run your build command
mvn clean package -DskipTests

echo "Build complete."
```

#### Option B — Execute Windows Batch Command (Windows agents)

```bat
echo Starting build...
echo Branch: %GIT_BRANCH%
mvn clean package -DskipTests
echo Build complete.
```

#### Option C — Invoke top-level Maven targets

| Field | Value |
|-------|-------|
| Maven Version | Select the Maven installation configured in Global Tool Configuration |
| Goals | `clean package` |
| POM | `pom.xml` (or path to your POM file) |

#### Option D — Invoke Gradle script

| Field | Value |
|-------|-------|
| Use Gradle Wrapper | ✅ Recommended if `gradlew` is in the repo |
| Tasks | `clean build` |

> 💡 You can **chain multiple build steps** — Jenkins executes them in order. If any step fails, the build is marked as failed and subsequent steps are skipped.

---

### Step 7 — Add Post-Build Actions

Post-build actions run **after** all build steps complete — regardless of success or failure (unless configured otherwise).

Click **Add post-build action** and choose:

#### Archive Artifacts

Store build output files so they're accessible from the Jenkins job page.

| Field | Value |
|-------|-------|
| Files to archive | `target/*.jar` or `target/*.war` |
| Only if build succeeds | ✅ Recommended |

#### Publish JUnit Test Results

If your build runs unit tests, publish the results for a visual report.

| Field | Value |
|-------|-------|
| Test report XMLs | `target/surefire-reports/*.xml` |

#### Email Notification

Send build status notifications to your team.

| Field | Value |
|-------|-------|
| Recipients | `team@yourcompany.com` |
| Send email for every broken build | ✅ |
| Send email when build is fixed | ✅ |

#### Trigger Downstream Jobs

Chain this job to another Jenkins job after completion.

| Field | Value |
|-------|-------|
| Projects to build | Name of the next Jenkins job |
| Trigger only if build is stable | ✅ Recommended |

---

### Step 8 — Save and Run

1. Click **Save** at the bottom of the configuration page
2. Click **Build Now** from the job's main page
3. A new entry appears under **Build History** — click on it
4. Click **Console Output** to follow the live build log

A successful build ends with:
```
Finished: SUCCESS
```

A failed build ends with:
```
Finished: FAILURE
```

---

## 🗂️ Understanding the Configuration Tabs

Here's a quick reference for every section of the Freestyle job configuration page:

| Section | What It Controls |
|---------|-----------------|
| **General** | Job description, build retention, display name, concurrent builds |
| **Source Code Management** | Git/SVN repo URL, credentials, branch, checkout behavior |
| **Build Triggers** | What starts the job — webhook, cron, poll SCM, upstream job |
| **Build Environment** | Workspace cleanup, timestamps, credential injection, tool versions |
| **Build Steps** | The actual commands or goals that execute during the build |
| **Post-build Actions** | Artifact archiving, test reports, notifications, downstream jobs |

---

## 💻 Common Build Step Examples

### Run a Python script

```bash
pip install -r requirements.txt
python run_tests.py
```

### Build a Docker image

```bash
docker build -t my-app:$BUILD_NUMBER .
docker tag my-app:$BUILD_NUMBER myregistry/my-app:latest
```

### Run a Node.js application build

```bash
npm install
npm run build
npm test
```

### Deploy to a remote server via SSH

```bash
scp -i ~/.ssh/deploy_key target/app.jar ubuntu@10.0.0.1:/opt/app/
ssh -i ~/.ssh/deploy_key ubuntu@10.0.0.1 "sudo systemctl restart app"
```

---

## 🛠️ Troubleshooting

### Build fails immediately — workspace issue

**Error:**
```
ERROR: No such file or directory
```
**Fix:** Enable **Delete workspace before build starts** in Build Environment, or check that your repository was cloned correctly by reviewing the console output for SCM errors.

---

### Git credentials rejected

**Error:**
```
remote: Repository not found.
fatal: repository 'https://github.com/...' not found
```
**Fix:** Go to **Manage Jenkins → Credentials** and verify your GitHub PAT is correct and has `repo` scope. Update the credential and re-save the job.

---

### Maven command not found

**Error:**
```
mvn: command not found
```
**Fix:** Use **Invoke top-level Maven targets** as your build step instead of a shell command, and ensure Maven is configured under **Manage Jenkins → Global Tool Configuration**.

---

### Build always shows as unstable (yellow)

**Cause:** JUnit test results are published but some tests are failing — Jenkins marks the build unstable rather than failed.
**Fix:** Fix the failing tests, or if test failures should mark the build as failed, adjust the **JUnit publisher** threshold settings.

---

## 📌 Summary

| Configuration Step | What Was Set Up |
|-------------------|----------------|
| ⚙️ General | Description, build retention, node restriction |
| 🔗 SCM | GitHub repository, credentials, branch |
| ⏰ Build Triggers | Webhook, cron schedule, or poll SCM |
| 🧹 Build Environment | Workspace cleanup, timestamps, credentials |
| 🏗️ Build Steps | Shell script, Maven goals, or Gradle tasks |
| 📦 Post-build Actions | Artifact archiving, test reports, email notifications |

---

<div align="center">

*Part of the [Jenkins Zero to Hero](../README.md) course*

</div>
