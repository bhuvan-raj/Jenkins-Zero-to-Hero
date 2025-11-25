
# **Building in Jenkins Using Maven – Study Notes**

<img src="https://github.com/bhuvan-raj/Jenkins-Zero-to-Hero/blob/main/assets/maven.jpg" alt="Banner" />


Maven is a popular **Java build automation tool**. Integrating Maven with Jenkins allows **automated compilation, testing, packaging, and deployment** in CI/CD pipelines.

---

## **1. Prerequisites**

1. **Jenkins Controller**

   * Installed and running
   * Access to Jenkins dashboard

2. **Jenkins Plugins**

   * **Maven Integration Plugin** (mandatory)
   * Optional: Git plugin (for source control), Pipeline plugin

3. **Java and Maven**

   * Install JDK on Jenkins controller or agent nodes
   * Install Maven on the system or configure Maven installation in Jenkins

4. **Project**

   * Maven project with `pom.xml`
   * SCM repository (Git, SVN, etc.)

---

## **2. Jenkins Maven Setup**

### **Step 1: Configure Maven in Jenkins**

1. Go to **Manage Jenkins → Global Tool Configuration**
2. Under **Maven**, click **Add Maven**
3. Configure:

   * **Name**: e.g., `Maven-3.9`
   * **Install automatically**: Check (optional, Jenkins downloads Maven automatically)
   * Or provide **Maven home directory** if installed manually
4. Save configuration

### **Step 2: Configure JDK**

1. In the same **Global Tool Configuration**, under **JDK**, add:

   * **Name**: `JDK-17`
   * **JAVA_HOME path** (optional if auto-installed)
2. Save configuration

---

## **3. Creating a Maven Job in Jenkins**

### **Step 1: Create Job**

1. **New Item → Enter Job Name → Maven Project → OK**

### **Step 2: Source Code Management**

1. Choose **Git** or **SVN** as SCM
2. Provide:

   * Repository URL
   * Credentials if needed
   * Branch to build

### **Step 3: Build Triggers**

* Configure automatic build triggers:

  * **Poll SCM**: e.g., `H/5 * * * *` (every 5 minutes)
  * **GitHub hook trigger**: triggers build on commit
  * **Cron schedule** for nightly builds

### **Step 4: Build Settings**

1. **Root POM**: Path to `pom.xml` (e.g., `pom.xml` or `module/pom.xml`)
2. **Goals and options**: Specify Maven goals, e.g.:

   * `clean install` – clean workspace and build project
   * `package` – compile and package project into JAR/WAR
   * `test` – run unit tests
   * `verify` – run integration tests
3. **Advanced Options** (optional):

   * Maven options (`-X` for debug)
   * JVM options (`-DskipTests=true` to skip tests)
   * Environment variables

### **Step 5: Post-build Actions**

* **Archive artifacts**: Save JAR/WAR or other output
* **Publish JUnit test results**: `**/target/surefire-reports/*.xml`
* **Deploy artifacts**: Push to Nexus, Artifactory, or GitHub Packages
* **Trigger other jobs**: For deployment or downstream steps

### **Step 6: Save and Build**

1. Click **Save**
2. Click **Build Now**
3. Check **Console Output** for logs:

   * Maven goals executed
   * Compilation status
   * Test results

---

## **4. Using Maven in Jenkins Pipeline**

Instead of a freestyle Maven job, you can use **Jenkins Pipeline** with Maven commands.

### **Declarative Pipeline Example**

```groovy
pipeline {
    agent any
    tools {
        maven 'Maven-3.9' // Maven installation configured in Jenkins
        jdk 'JDK-17'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/example/repo.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }
    }
}
```

---

### **Pipeline Notes**

1. `tools` block automatically sets `PATH` to use configured Maven and JDK.
2. `sh` steps execute Maven commands on the selected agent.
3. `junit` step publishes test results to Jenkins UI.
4. Pipeline allows **parallel builds**, **dynamic agents**, and **multi-stage workflows**.

---

## **5. Best Practices for Maven Builds in Jenkins**

| Best Practice                                           | Reason                                                   |
| ------------------------------------------------------- | -------------------------------------------------------- |
| Use `clean` goal first                                  | Remove previous artifacts to avoid conflicts             |
| Archive artifacts                                       | Keep build outputs for future reference                  |
| Publish test results                                    | Track quality and CI feedback                            |
| Use environment-specific profiles                       | Handle dev/test/prod builds via Maven profiles           |
| Use pipeline instead of freestyle for complex workflows | Better scalability, maintainability, and code versioning |
| Use Jenkins agents for builds                           | Isolate environments and scale horizontally              |
| Configure Maven options like `-B` (batch mode)          | Avoid interactive prompts in CI                          |

---

## **6. Integration with Other Tools**

* **Artifact Repositories**: Nexus, Artifactory, GitHub Packages

  * Use Maven `deploy` goal to push artifacts
* **Testing Frameworks**: JUnit, TestNG, Surefire, Jacoco
* **Code Quality**: SonarQube, Checkstyle, PMD integration in Maven build
* **Notifications**: Email, Slack, Teams for build status

---

## **7. Debugging Maven Builds in Jenkins**

1. Enable **Debug mode**: `mvn -X clean install`
2. Check **Console Output** for:

   * Dependency resolution errors
   * Compilation errors
   * Test failures
3. Use **Build Environment variables**:

   * `WORKSPACE` – path of Jenkins workspace
   * `BUILD_NUMBER` – Jenkins build number
   * `JOB_NAME` – Jenkins job name

---

## **8. Summary**

* Maven builds in Jenkins can be managed using **Freestyle Maven Jobs** or **Pipelines**.
* Configure **Maven installation** and **JDK** in Jenkins.
* Key goals: `clean`, `install`, `package`, `test`, `deploy`.
* Integrate with **SCM, test reporting, artifact repositories**.
* Pipelines offer **better maintainability**, **parallel builds**, and **dynamic agents**.
* Secure credentials for repository access using Jenkins **Credentials Plugin**.

---
