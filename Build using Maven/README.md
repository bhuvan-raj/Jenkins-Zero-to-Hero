<img src="https://github.com/bhuvan-raj/Jenkins-Zero-to-Hero/blob/main/assets/maven.jpg" alt="Banner" />

<div align="center">

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

# Lab Note: Build & Push Java Artifact to GitHub Packages using Maven + Jenkins Freestyle

---

## Prerequisites

- Jenkins instance with the following plugins installed: **Maven Integration**, **GitHub**, **Credentials Binding**
- JDK and Maven configured in Jenkins → Manage Jenkins → Global Tool Configuration
- A Java Maven project hosted on GitHub
- A GitHub Personal Access Token (PAT) with `write:packages` and `read:packages` scopes

---

## Step 1 — Configure GitHub PAT in Jenkins Credentials

1. Go to **Jenkins → Manage Jenkins → Credentials → (global) → Add Credentials**
2. Fill in:
   - **Kind:** `Username with password`
   - **Username:** your GitHub username
   - **Password:** your GitHub PAT
   - **ID:** `github-packages-creds` *(you'll reference this later)*
3. Click **Save**

---

## Step 2 — Update `pom.xml` for GitHub Packages

Add the distribution management block so Maven knows where to publish:

```xml
<distributionManagement>
  <repository>
    <id>github</id>
    <name>GitHub Packages</name>
    <url>https://maven.pkg.github.com/YOUR_GITHUB_USERNAME/YOUR_REPO_NAME</url>
  </repository>
</distributionManagement>
```

Ensure your `pom.xml` has the correct artifact identifiers:

```xml
<groupId>com.yourcompany</groupId>
<artifactId>your-artifact</artifactId>
<version>1.0.0</version>
<packaging>jar</packaging>
```

> **Note:** The `<id>github</id>` here must exactly match the server `id` in `settings.xml` (Step 3).

---

## Step 3 — Create `settings.xml` on the Jenkins Server

Maven needs credentials at deploy time. Create the file at the Jenkins user's home directory:

```bash
mkdir -p /var/lib/jenkins/.m2
nano /var/lib/jenkins/.m2/settings.xml
```

Paste in the following content:

```xml
<settings>
  <servers>
    <server>
      <id>github</id>
      <username>${env.GITHUB_USERNAME}</username>
      <password>${env.GITHUB_TOKEN}</password>
    </server>
  </servers>
</settings>
```

Verify the file exists and is correctly named:

```bash
ls /var/lib/jenkins/.m2/
# Expected output: settings.xml
```

> **Common mistake:** Watch for filename typos like `settinga.xml`. Maven will throw a "file does not exist" error if the name is wrong. Always verify with `ls`.

---

## Step 4 — Create a Jenkins Freestyle Job

1. Go to **Jenkins → New Item → Freestyle Project**, give it a name, click **OK**

2. **Source Code Management:**
   - Select **Git**
   - Enter your repo URL: `https://github.com/YOUR_USERNAME/YOUR_REPO.git`
   - Add credentials — select `github-packages-creds`
   - Set branch to `*/main` (or your target branch)

3. **Build Triggers** *(optional but recommended):*
   - **GitHub hook trigger for GITScm polling** — for webhook-based triggers, or
   - **Poll SCM** — use cron like `H/5 * * * *`

---

## Step 5 — Bind Credentials as Environment Variables

1. Under **Build Environment**, check **Use secret text(s) or file(s)**
2. Click **Add → Username and password (separated)**
3. Fill in:
   - **Username Variable:** `GITHUB_USERNAME`
   - **Password Variable:** `GITHUB_TOKEN`
   - **Credentials:** select `github-packages-creds`

This injects your PAT safely as environment variables during the build so `settings.xml` can reference them.

---

## Step 6 — Configure the Build Step

1. Under **Build**, click **Add build step → Invoke top-level Maven targets**
2. Set:
   - **Maven Version:** select the Maven installation from Global Tools
   - **Goals:**
     ```
     clean deploy -s /var/lib/jenkins/.m2/settings.xml
     ```

> **Important:** Always use the full absolute path `/var/lib/jenkins/.m2/settings.xml`.  
> Do **not** use `~/.m2/settings.xml` or `$HOME/.m2/settings.xml` in the Maven targets field —  
> the tilde and variables are not shell-expanded in this build step and will cause a path error.

To skip tests:
```
clean deploy -DskipTests -s /var/lib/jenkins/.m2/settings.xml
```

---

## Step 7 — Save and Run the Job

1. Click **Save**, then **Build Now**
2. Monitor **Console Output** — a successful deploy will show:

```
[INFO] BUILD SUCCESS
```

Along with upload confirmation lines like:

```
Uploading to github: https://maven.pkg.github.com/YOUR_USERNAME/YOUR_REPO/...
Uploaded to github: https://maven.pkg.github.com/...
```

---

## Step 8 — Verify on GitHub

1. Navigate to your GitHub repository
2. Click **Packages** (right sidebar or top navigation)
3. Your artifact should be listed with the version defined in `pom.xml`

---

## Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `The specified user settings file does not exist` | Wrong path or filename typo (e.g. `settinga.xml`) | Run `ls /var/lib/jenkins/.m2/` and rename with `mv settinga.xml settings.xml` |
| Path resolves to workspace (e.g. `/var/lib/jenkins/workspace/deploy/~/.m2/...`) | Using `~` in Maven targets field — tilde is not expanded | Use full absolute path `/var/lib/jenkins/.m2/settings.xml` |
| `401 Unauthorized` | PAT missing `write:packages` scope or wrong credentials bound | Regenerate PAT with correct scopes; verify credential ID matches |
| `server id mismatch` | `id` in `settings.xml` doesn't match `id` in `pom.xml` `<distributionManagement>` | Ensure both are exactly `github` (or whatever you chose) |
| `409 Conflict` — artifact already exists | GitHub Packages does not allow overwriting a published version | Bump the version in `pom.xml` |
| Build succeeds but package not visible | Wrong repo URL in `<distributionManagement>` | Double-check `YOUR_GITHUB_USERNAME` and `YOUR_REPO_NAME` in the URL |

---

## Key Reminders

- The `-s` flag in **Invoke top-level Maven targets** does **not** perform shell expansion. Always use the full absolute path.
- The `<id>` tag in `settings.xml` and `pom.xml` must be **identical**.
- GitHub Packages is **immutable** — you cannot overwrite an existing version. Always increment your version for each publish.
- If unsure of the Jenkins home directory, run: `getent passwd jenkins | cut -d: -f6`

