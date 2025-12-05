

# **Multibranch Pipeline in Jenkins — An In-Depth Explanation**

A **Multibranch Pipeline** in Jenkins is a specialized job type that automatically discovers, manages, and executes pipelines for every branch (and PR) in a source control repository. It eliminates the need to manually create separate jobs for each branch and enables modern CI/CD workflows aligned with Git-based development practices.

---

## **1. What is a Multibranch Pipeline?**

A Multibranch Pipeline is a Jenkins project that:

* Scans a **source code repository** (such as GitHub, Bitbucket, or GitLab) for all branches.
* Automatically detects branches containing a `Jenkinsfile`.
* Creates and configures individual pipeline jobs for each detected branch.
* Triggers builds based on branch-specific changes.
* Manages lifecycle of discovered branches (creation/deletion).

It is fully automated and designed for teams following Git workflows such as:

* Git Feature Branching
* GitFlow
* Trunk-based Development
* PR-based development

---

# **2. Why Multibranch Pipeline? — Key Benefits**

### **a. Automation of Branch Jobs**

No need to manually create jobs for `dev`, `staging`, `feature/*`, `hotfix/*`, etc.
Jenkins dynamically provisions them.

### **b. Separate Pipelines Per Branch**

Each branch can have:

* Its own `Jenkinsfile`
* Different build logic
* Independent build history
* Separate artifacts and test reports



# **🔬 LAB: Creating Branches from GitHub (Web UI) for Multibranch Pipeline**

This lab will teach you how to:

1. Create a GitHub repository
2. Create branches **directly from the GitHub UI** (no CLI required)
3. Add a different Jenkinsfile to each branch
4. Prepare everything for Jenkins Multibranch Pipeline scanning

This is ideal when teaching CI/CD concepts without using local Git commands.

---

# **🧰 Step 1 — Create a New GitHub Repository**

1. Log in to GitHub
2. Click **New Repository**
3. Repository name:
   **multibranch-demo**
4. Select **Public**
5. Add a README → ✔️ check *“Add a README file”*
6. Click **Create Repository**

Your repository is now created with the `main` branch.

---

# **🌿 Step 2 — Create Branches from GitHub**

GitHub allows branch creation directly from the UI.

## ✔️ Create Branch 1 — main (already exists)

This is created automatically.

---

## ✔️ Create Branch 2 — dev

1. Go to the repository homepage
2. Click the **Branch dropdown** (top-left showing `main`)
3. In the text box, type:

   ```
   dev
   ```
4. Click **Create branch: dev from 'main'**

GitHub now creates the `dev` branch.

---

## ✔️ Create Branch 3 — feature-ui

1. Click the **Branch dropdown** again
2. Type:

   ```
   feature-ui
   ```
3. Click **Create branch: feature-ui from 'main'**

Now you have:

```
main
dev
feature-ui
```

---

# **📁 Step 3 — Add Jenkinsfile to Each Branch (using GitHub editor)**

Each branch must have its own Jenkinsfile so Jenkins Multibranch Pipeline can discover it.

---

## ✔️ Add Jenkinsfile to main

1. Go to **Branch dropdown → main**
2. Click **Add file → Create new file**
3. File name:

   ```
   Jenkinsfile
   ```
4. Paste the following:

```
pipeline {
    agent any

    stages {
        stage('Main Branch Stage') {
            steps {
                echo "This is the MAIN branch build."
                echo "Executing main branch pipeline..."
            }
        }
    }
}
```

5. Scroll down → **Commit directly to the main branch**

---

## ✔️ Add Jenkinsfile to dev branch

1. Switch to branch:
   **Branch dropdown → dev**
2. Click **Add file → Create new file**
3. File name:

   ```
   Jenkinsfile
   ```
4. Paste:

```
pipeline {
    agent any

    stages {
        stage('Dev Branch Stage') {
            steps {
                echo "This is the DEV branch build."
                echo "Development environment pipeline executing..."
            }
        }
    }
}
```

5. Commit directly to **dev**.

---

## ✔️ Add Jenkinsfile to feature-ui branch

1. Switch to **feature-ui** branch
2. Click **Add file → Create new file**
3. File name:

   ```
   Jenkinsfile
   ```
4. Paste:

```
pipeline {
    agent any

    stages {
        stage('Feature UI Stage') {
            steps {
                echo "This is the FEATURE-UI branch build."
                echo "Running pipeline for UI feature development..."
            }
        }
    }
}
```

5. Commit directly to **feature-ui**.

---

# **🎯 Step 4 — Verify All Branches Have Jenkinsfiles**

From GitHub:

1. Click **Branch dropdown**
2. Select each branch (main, dev, feature-ui)
3. Ensure each branch shows a **Jenkinsfile** at the root

This ensures Jenkins will detect all branches.

---

# **🔗 Step 5 — Use These Branches in Jenkins Multibranch Pipeline**

Once these branches are created and contain Jenkinsfiles, move to Jenkins:

1. Create a **Multibranch Pipeline** job
2. Add GitHub as **Branch Source**
3. Add repo URL:

   ```
   https://github.com/<your-username>/multibranch-demo
   ```
4. Save and run **Scan Multibranch Pipeline Now**

Jenkins will automatically discover:

* main
* dev
* feature-ui

Each branch will run its respective Jenkinsfile.

---

# **🎉 Lab Completed Successfully**

You now have:

✔ Branches created directly on GitHub
✔ Jenkinsfiles added using GitHub’s online editor
✔ Repository ready for Jenkins Multibranch Pipeline
✔ Zero local Git usage (completely browser-based workflow)

---
