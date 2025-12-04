# **📘 Jenkins Pipeline Lab: Build & Deploy Maven Package to GitHub Packages**

This lab demonstrates how to use **Jenkins Pipeline** to:

1. Build a Java application using Maven
2. Use a managed `settings.xml` through Config File Management
3. Authenticate with GitHub using secure Jenkins credentials
4. Deploy your artifact to **GitHub Packages (Maven Registry)**

---

# **1. Prerequisites**

### **Jenkins Requirements**

* Jenkins installed
* Maven installed and configured (`Manage Jenkins → Global Tool Configuration`)
* JDK installed and configured

### **GitHub Requirements**

* A GitHub repository
* A Personal Access Token (PAT) with:

  * `write:packages`
  * `read:packages`
  * `repo`

### **Jenkins Plugins Required**

* Pipeline
* Git
* Config File Provider
* GitHub Integration
* Maven Integration

---

# **2. Create GitHub PAT Credential in Jenkins**

1. Go to: **Manage Jenkins → Credentials**
2. Add a **Username + Password** credential

   * **Username:** Your GitHub username
   * **Password:** GitHub PAT
   * **ID:** `github-packages-cred`

---

# **3. Add Maven settings.xml Using Config File Management**

1. Go to **Manage Jenkins → Managed files**
2. Click **Add a new Config**
3. Choose **Global Maven Settings File**
4. Use:

   * **Name:** `maven-github-settings`
   * **ID:** `maven-github-settings`
5. Paste the following:

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0">
  <servers>
    <server>
      <id>github</id>
      <username>${env.GH_USER}</username>
      <password>${env.GH_TOKEN}</password>
    </server>
  </servers>
</settings>
```

Click **Save**.

---

# **4. Create the Java Application**

### **App.java**

```
src/main/java/com/example/App.java
```

```java
package com.example;

public class App {
    public static void main(String[] args) {
        System.out.println("Hello Jenkins CI/CD!");
    }
}
```

---

# **5. Configure Maven to Deploy to GitHub Packages**

### **pom.xml**

Replace `YOUR_USERNAME` and `YOUR_REPO` with real values.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>jenkins-demo</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <name>Jenkins GitHub Packages Demo</name>

    <distributionManagement>
        <repository>
            <id>github</id>
            <name>GitHub Packages</name>
            <url>https://maven.pkg.github.com/YOUR_USERNAME/YOUR_REPO</url>
        </repository>
    </distributionManagement>

</project>
```

---

# **6. Create Jenkins Pipeline Job**

### Steps:

1. Go to **Jenkins → New Item**
2. Choose **Pipeline**
3. Select **Pipeline Script from SCM**
4. Enter your GitHub repository URL
5. Point to your `Jenkinsfile`

---

# **7. Create Jenkinsfile**

```groovy
pipeline {
    agent any

    environment {
        GITHUB_CREDS = credentials('github-packages-cred')
        JAVA_HOME = tool name: 'jdk11'
        MAVEN_HOME = tool name: 'maven3'
        PATH = "${JAVA_HOME}/bin:${PATH}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build & Deploy') {
            steps {
                configFileProvider([configFile(fileId: 'maven-github-settings', variable: 'MAVEN_SETTINGS')]) {
                    sh """
                        export GH_USER=${GITHUB_CREDS_USR}
                        export GH_TOKEN=${GITHUB_CREDS_PSW}

                        ${MAVEN_HOME}/bin/mvn -s $MAVEN_SETTINGS -B clean package
                        ${MAVEN_HOME}/bin/mvn -s $MAVEN_SETTINGS -B deploy
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Build and deployment completed successfully."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
```

---

# **8. Run the Pipeline**

After configuring the job:

1. Click **Build Now**
2. Jenkins will:

   * Checkout the repo
   * Build your project
   * Package the JAR
   * Use the managed settings.xml for authentication
   * Deploy to GitHub Packages

---

# **9. Verify Package on GitHub**

Go to:

```
GitHub → Your Repository → Packages
```

You should see:

* `jenkins-demo`
* Version `1.0.0`
* JAR + POM files

---

* Jenkins Pipeline
* GitHub Packages
* Maven build & deployment
* Secure credential management
* Config File Management

