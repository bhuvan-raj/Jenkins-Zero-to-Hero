# **Building in Jenkins Using Maven – Study Notes**

<img src="https://github.com/bhuvan-raj/Jenkins-Zero-to-Hero/blob/main/assets/maven.jpg" alt="Banner" />


Maven is a popular **Java build automation tool**. Integrating Maven with Jenkins allows **automated compilation, testing, packaging, and deployment** in CI/CD pipelines.

---

# **1. Lab Objective**

Learners will configure a Jenkins Freestyle project that:

* Pulls a Maven project from GitHub
* Executes a Maven build (`clean package`)
* Archives generated artifacts
* Shows build output through Jenkins console

---

# **2. Prerequisites**

Before starting the Jenkins configuration:

* Jenkins server is running
* Java is installed on the Jenkins machine
* Maven is installed or configured through Jenkins
* Git is installed
* A valid GitHub repository containing a Maven project

---

# **3. Configure Maven in Jenkins**

### **Step 1: Open Global Tool Configuration**

1. From Jenkins Dashboard → **Manage Jenkins**
2. Click **Global Tool Configuration**

### **Step 2: Add Maven**

1. Scroll to **Maven** section
2. Click **Add Maven**
3. Fill in:

   * **Name:** `Maven-3.9`
   * **Install automatically:** ✓ (checked)
4. Save settings

This makes Maven available inside Freestyle jobs.

---

# **4. Create a Freestyle Project**

### **Step 3: Start a New Job**

1. From Jenkins Dashboard → **New Item**
2. Job Name:

   ```
   maven-build-job
   ```
3. Select **Freestyle project**
4. Click **OK**

---

# **5. Configure Source Code Management (SCM)**

### **Step 4: Connect GitHub Repository**

Inside the project configuration page:

1. Go to **Source Code Management**

2. Select **Git**

3. Enter the repository URL:

   Example:

   ```
   https://github.com/<username>/<repository>.git
   ```

4. For private repos → Add credentials

5. Under **Branches to build**, set:

   ```
   */main
   ```

   (or the branch your project uses)

This ensures Jenkins retrieves the project files during the build.

---

# **6. Add Maven Build Step**

### **Step 5: Add Build Instruction**

1. Scroll to **Build** section
2. Click **Add build step** → **Invoke top-level Maven targets**

Configure:

* **Maven Version:**
  Select `Maven-3.9` (the one added earlier)

* **Goals:**

  ```
  clean package
  ```

Explanation:

* `clean` removes previous builds
* `package` produces the JAR/WAR file

---

# **7. Archive Build Artifacts**

### **Step 6: Add Post-build Action**

1. Scroll to **Post-build Actions**
2. Click **Add post-build action**
3. Select **Archive the artifacts**
4. In the **Files to archive** field, enter:

   ```
   target/*.jar
   ```

   or

   ```
   target/*.war
   ```

This allows Jenkins to store and display the artifact for download after every successful build.

---

# **8. Build the Job**

### **Step 7: Save and Run**

1. Click **Save**
2. Click **Build Now**

This triggers the first execution.

---

# **9. Validate Build Output**

### **Step 8: Check Console Output**

1. Click on the build number (e.g., **#1**)
2. Click **Console Output**

Expected behavior:

* Jenkins pulls source code from GitHub
* Maven downloads dependencies
* Compilation happens
* Packaging completes
* Output ends with:

  ```
  BUILD SUCCESS
  ```

### **Step 9: Verify Artifacts**

1. Open the same build page
2. Scroll to **Artifacts** section
3. You should see:

   ```
   your-app-1.0.0.jar
   ```
4. You may download it to verify locally.

---

# **10. Troubleshooting (Very Important for Teaching)**

### **Issue 1: Git not installed**

Error:

```
git: command not found
```

Fix:

```bash
sudo dnf install git
```

### **Issue 2: Maven not found**

Error:

```
No such Maven installation found
```

Fix:
Reconfigure in *Global Tool Configuration*.

### **Issue 3: Build fails because pom.xml missing**

Fix:
Ensure the GitHub project contains a valid Maven project structure.

---

# **11. Lab Summary**

In this lab, learners:

* Integrated Jenkins with GitHub
* Configured Maven inside Jenkins
* Created a Freestyle job
* Ran Maven build (`clean package`)
* Archived build outputs
* Verified successful build execution

