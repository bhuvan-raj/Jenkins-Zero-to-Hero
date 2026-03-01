<img src="https://github.com/bhuvan-raj/Jenkins-Zero-to-Hero/blob/main/assets/maven.jpg" alt="Banner" />


![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Maven](https://img.shields.io/badge/Apache%20Maven-C71A36?style=for-the-badge&logo=apache-maven&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)

# Building with Maven in Jenkins

> **Pull from GitHub. Compile. Test. Package. Archive. All automated.**

</div>

---

## 📋 Table of Contents

- [Lab Objective](#-lab-objective)
- [Prerequisites](#-prerequisites)
- [Step 1 — Configure Maven in Jenkins](#step-1--configure-maven-in-jenkins)
- [Step 2 — Create a Freestyle Project](#step-2--create-a-freestyle-project)
- [Step 3 — Configure Source Code Management](#step-3--configure-source-code-management-scm)
- [Step 4 — Add the Maven Build Step](#step-4--add-the-maven-build-step)
- [Step 5 — Archive Build Artifacts](#step-5--archive-build-artifacts)
- [Step 6 — Build and Validate](#step-6--build-and-validate)
- [Troubleshooting](#-troubleshooting)
- [Lab Summary](#-lab-summary)

---

## 🎯 Lab Objective

In this lab, you will configure a **Jenkins Freestyle project** that:

- Pulls a Maven project from a GitHub repository
- Executes a Maven build using the `clean package` goal
- Archives the generated JAR/WAR artifact
- Displays full build output through the Jenkins console

By the end, you will have a fully working automated Java build pipeline in Jenkins.

---

## ✅ Prerequisites

Before starting, ensure the following are in place on your Jenkins machine:

| Requirement | Details |
|-------------|---------|
| Jenkins | Running and accessible via browser |
| Java | Installed on the Jenkins server (`java -version`) |
| Git | Installed on the Jenkins server (`git --version`) |
| Maven | Installed locally **or** configured via Jenkins Global Tool Configuration |
| GitHub Repository | A valid Maven project with a `pom.xml` at the root |

---

## Step 1 — Configure Maven in Jenkins

Before Jenkins can run Maven builds, it needs to know where Maven is. You can have Jenkins install it automatically.

1. From the Jenkins Dashboard, go to **Manage Jenkins → Global Tool Configuration**
2. Scroll down to the **Maven** section
3. Click **Add Maven** and fill in:

| Field | Value |
|-------|-------|
| Name | `Maven-3.9` |
| Install automatically | ✅ Checked |

4. Click **Save**

Maven is now available as a build tool for all jobs on this Jenkins instance.

---

## Step 2 — Create a Freestyle Project

1. From the Jenkins Dashboard, click **New Item**
2. Enter a job name — e.g., `maven-build-job`
3. Select **Freestyle project**
4. Click **OK**

---

## Step 3 — Configure Source Code Management (SCM)

Inside the project configuration page, connect Jenkins to your GitHub repository:

1. Scroll to the **Source Code Management** section
2. Select **Git**
3. Enter your repository URL:

```
https://github.com/<your-username>/<your-repository>.git
```

4. For **private repositories**, click **Add Credentials** and provide your GitHub username and personal access token
5. Under **Branches to build**, set the branch:

```
*/main
```

> 💡 Change `main` to `master` or your feature branch name if needed.

Jenkins will now clone this repository every time the job runs.

---

## Step 4 — Add the Maven Build Step

Now tell Jenkins what to do with the code once it's pulled:

1. Scroll to the **Build** section
2. Click **Add build step → Invoke top-level Maven targets**
3. Configure:

| Field | Value |
|-------|-------|
| Maven Version | `Maven-3.9` (the one configured in Step 1) |
| Goals | `clean package` |

**What these Maven goals do:**

- `clean` — removes any previously generated build files from the `target/` directory
- `package` — compiles the source code, runs tests, and packages the output into a JAR or WAR file

---

## Step 5 — Archive Build Artifacts

After a successful build, archive the output so it's accessible directly from the Jenkins job page:

1. Scroll to **Post-build Actions**
2. Click **Add post-build action → Archive the artifacts**
3. In the **Files to archive** field, enter:

```
target/*.jar
```

For WAR-based projects (e.g., Spring MVC web apps):

```
target/*.war
```

Jenkins will now store and display the artifact after every successful build, making it available for download.

---

## Step 6 — Build and Validate

### Trigger the Build

1. Click **Save**
2. Click **Build Now**

Jenkins will queue the job and begin execution immediately.

---

### Check the Console Output

1. Click on the build number (e.g., **#1**) in the **Build History** panel
2. Click **Console Output**

A successful build will show Maven downloading dependencies, compiling, testing, and packaging the app. The output should end with:

```
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  12.345 s
[INFO] Finished at: 2026-03-01T10:00:00Z
```

---

### Verify the Archived Artifact

1. Navigate back to the build page (e.g., **Build #1**)
2. Scroll to the **Artifacts** section
3. You should see something like:

```
your-app-1.0.0.jar
```

Click the file to download and verify it locally if needed.

---

## 🛠️ Troubleshooting

### Git not found on the agent

**Error:**
```
git: command not found
```

**Fix — Install Git on the Jenkins server:**
```bash
# Amazon Linux / RHEL / CentOS
sudo dnf install git -y

# Ubuntu / Debian
sudo apt install git -y
```

---

### Maven installation not found

**Error:**
```
No such Maven installation found: Maven-3.9
```

**Fix:** Go back to **Manage Jenkins → Global Tool Configuration** and verify the Maven installation name matches exactly what is set in the job (`Maven-3.9`). Re-save both.

---

### Build fails — pom.xml not found

**Error:**
```
[ERROR] The goal you specified requires a project to execute but there is no POM in this directory.
```

**Fix:** Ensure the GitHub repository contains a valid Maven project structure with `pom.xml` at the root:

```
your-repo/
├── src/
│   ├── main/java/
│   └── test/java/
└── pom.xml        ← Must be here
```

---

### Build fails — compilation errors

**Fix:** Check the **Console Output** for the specific Java compile error. Fix the source code in your repository and re-trigger the build. Jenkins will pull the latest code and retry.

---

## 📌 Lab Summary

| Step | What Was Done |
|------|---------------|
| ⚙️ Configure Maven | Added Maven 3.9 via Global Tool Configuration |
| 📁 Create Job | Created a Freestyle project called `maven-build-job` |
| 🔗 Connect GitHub | Configured Git SCM with repo URL and branch |
| 🏗️ Add Build Step | Added `clean package` Maven goal |
| 📦 Archive Artifacts | Set `target/*.jar` as the archived artifact |
| ▶️ Run & Verify | Triggered build, verified `BUILD SUCCESS` and artifact download |

---

<div align="center">

*Part of the [Jenkins Zero to Hero](../README.md) course*

</div>
