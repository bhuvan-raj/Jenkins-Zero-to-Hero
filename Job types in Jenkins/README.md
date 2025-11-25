
# **Jenkins Job Types – In-Depth Guide**

Jenkins supports multiple types of jobs (projects), each suited for different automation scenarios. A **job** is basically a task or pipeline Jenkins executes—like building code, running tests, or deploying an application.

---

# **1. Freestyle Project**

The Freestyle project is the most basic and flexible Jenkins job type.

### **Use Case**

* Simple builds
* Running scripts (shell, batch)
* Integrating with SCM (Git, SVN)
* Triggering builds manually or via SCM/webhooks

### **Steps to Create a Freestyle Job**

1. Go to **Jenkins Dashboard → New Item**
2. Enter a **Job name**
3. Select **Freestyle project**
4. Click **OK**
5. Configure:

   * **General tab:** Description, discard old builds, restrict where the job runs
   * **Source Code Management (SCM):** Git/ SVN repository, credentials, branches
   * **Build Triggers:** Poll SCM, GitHub webhook, schedule (CRON)
   * **Build Environment:** Clean workspace before build, environment variables
   * **Build Steps:** Add build steps like:

     * Execute shell (Linux) / Execute Windows batch command
     * Invoke Ant, Maven, Gradle
   * **Post-build Actions:** Archive artifacts, email notifications, trigger other jobs
6. Click **Save**
7. Click **Build Now** to test

---

# **2. Maven Project**

Specifically for Java projects using **Maven** as the build tool.

### **Use Case**

* Java applications
* Automatic dependency management
* Integration with Maven goals (compile, test, package)

### **Steps to Create a Maven Job**

1. Install **Maven Integration Plugin** (if not already installed)
2. **New Item → Enter Job name → Maven project → OK**
3. Configure:

   * **Source Code Management:** Git/SVN repo
   * **Build Triggers:** Poll SCM, GitHub hook, timer
   * **Build Settings:**

     * Root POM path (pom.xml location)
     * Goals (e.g., `clean install`)
     * Options (skip tests, build options)
   * **Post-build Actions:** Archive artifacts, run other jobs
4. Save and run **Build Now**

---

# **3. Pipeline Job**

The **Pipeline** job uses **Jenkinsfile** (code-based configuration). It is the modern, preferred way for CI/CD.

### **Use Case**

* Complex workflows
* Multi-stage CI/CD pipelines
* Parallel builds
* Version-controlled pipeline code

### **Steps to Create a Pipeline Job**

1. **New Item → Pipeline → OK**
2. Configure:

   * **General Tab:** Description, discard old builds
   * **Build Triggers:** SCM webhook, cron schedule
   * **Pipeline Definition:** Two main options:

     * **Pipeline script:** Write Groovy code directly in Jenkins
     * **Pipeline script from SCM:** Store Jenkinsfile in repository
3. Sample **Declarative Pipeline Script**:

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

---

# **4. Multibranch Pipeline**

Automates pipeline creation for **multiple branches** in a repository.

### **Use Case**

* Git repositories with feature branches
* Automatically create pipelines per branch
* Supports Pull Request builds

### **Steps to Create a Multibranch Pipeline**

1. Install **Multibranch Pipeline Plugin**
2. **New Item → Multibranch Pipeline → OK**
3. Configure:

   * **Branch Sources:** Git, GitHub, Bitbucket
   * **Credentials:** Repo access
   * **Discover branches and PRs:** Enable automatic detection
   * **Build Configuration:** Specify `Jenkinsfile` path
4. Jenkins automatically scans branches and creates individual jobs per branch

---

# **5. GitHub Organization Job**

Automatically discovers all repositories in a GitHub organization.

### **Use Case**

* Large GitHub organizations
* Auto-manage jobs for new repositories
* Multibranch pipeline support

### **Steps to Create a GitHub Organization Job**

1. **New Item → GitHub Organization → OK**
2. Configure:

   * **Owner:** GitHub organization name
   * **Credentials:** GitHub token
   * **Repository discovery:** Filter by patterns (optional)
   * **Build Configuration:** `Jenkinsfile` location
3. Jenkins scans repositories and creates pipelines for each repository

---

# **6. Folder Job**

Folders are **containers for organizing jobs**.

### **Use Case**

* Organize multiple jobs
* Apply folder-level credentials and permissions
* Group related projects

### **Steps to Create a Folder**

1. **New Item → Folder → OK**
2. Enter **Folder name**
3. Add jobs inside folder
4. Apply folder-specific credentials or permissions

---

# **7. External Job**

Monitors jobs executed **outside Jenkins**.

### **Use Case**

* Legacy builds
* Cron jobs
* External scripts

### **Steps to Create an External Job**

1. **New Item → External Job → OK**
2. Configure:

   * Description
   * Location (hostname or path of external job)
   * Build triggers (poll external status)
3. Jenkins monitors execution and captures build results

---

# **8. Matrix/Multiconfiguration Job**

Runs the same job **across multiple environments** or configurations.

### **Use Case**

* Test across multiple OS/Java versions
* Build for multiple platforms simultaneously

### **Steps to Create a Matrix Job**

1. **New Item → Multiconfiguration project → OK**
2. Configure:

   * **General → Name**
   * **Axes:** Define variables (OS, JDK, Node labels)
   * **Build Steps:** Same for all configurations
   * **Post-build Actions**
3. Jenkins automatically creates combinations and executes builds in parallel across agents

---

# **9. Summary Table of Jenkins Job Types**

| Job Type                    | Use Case / Notes                                     |
| --------------------------- | ---------------------------------------------------- |
| Freestyle Project           | Simple builds, scripts, basic SCM integration        |
| Maven Project               | Java projects with Maven build automation            |
| Pipeline                    | Code-defined CI/CD, complex workflows                |
| Multibranch Pipeline        | Automatic branch detection & pipeline per branch     |
| GitHub Organization         | Auto-manage pipelines for all repositories in an org |
| Folder                      | Organize multiple jobs                               |
| External Job                | Monitor non-Jenkins jobs                             |
| Matrix / Multiconfiguration | Parallel execution across multiple environments      |

---
