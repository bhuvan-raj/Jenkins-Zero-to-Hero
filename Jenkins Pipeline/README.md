
# **Jenkins Pipeline – In-Depth Study Note**

## **1. Introduction to Jenkins Pipeline**

A **Jenkins Pipeline** is a suite of plugins that supports the implementation of **continuous integration and continuous delivery (CI/CD)** workflows.
It allows you to define the entire build, test, and deploy process as **code** inside a file called **Jenkinsfile**, stored in your source control repository.

### **Why Pipeline?**

* Provides **Pipeline as Code**
* Highly **reliable**, **repeatable**, and **version-controlled**
* Supports **complex workflows**
* **Parallel execution** capabilities
* Better visualization of stages/steps

---

# **2. Pipeline as Code – Jenkinsfile**

A **Jenkinsfile** is a text file that contains pipeline definitions written in:

* **Declarative Pipeline Syntax**
* **Scripted Pipeline Syntax**

Stored inside your project repository (e.g., GitHub, GitLab), it ensures CI/CD logic is versioned with the application code.

---

# **3. Types of Jenkins Pipelines**

## **A. Declarative Pipeline (Most Preferred)**

The **recommended** and **modern** pipeline syntax used by most companies today.

### **Key Characteristics**

* Simpler, more readable, and structured
* Strict and opinionated syntax
* Built-in error detection and validation
* Best for teams and large projects

### **Basic Example**

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo "Building the application..."
            }
        }
        stage('Test') {
            steps {
                echo "Running tests..."
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying..."
            }
        }
    }
}
```

---

## **B. Scripted Pipeline**

The older and more flexible pipeline type.
Written using **Groovy** and allows full programmatic control.

### **Key Characteristics**

* Very flexible and powerful
* Fully Groovy-based scripting
* Ideal for complex logic
* Less readable compared to declarative

### **Basic Example**

```groovy
node {
    stage('Build') {
        echo "Building the application..."
    }
    stage('Test') {
        echo "Running tests..."
    }
    stage('Deploy') {
        echo "Deploying..."
    }
}
```

---

# **4. Declarative vs Scripted Pipeline**

| Feature           | Declarative      | Scripted                |
| ----------------- | ---------------- | ----------------------- |
| Readability       | High             | Medium                  |
| Syntax Strictness | High             | Low                     |
| Flexibility       | Medium           | Very High               |
| Learning Curve    | Easy             | Hard (Groovy-based)     |
| Industry Use      | Highly Preferred | Used for advanced cases |
| Error Handling    | Built-in         | Manual                  |

### **Industry Preference**

Most companies prefer **Declarative Pipeline**, because:

* It's more maintainable
* Easier for teams to collaborate
* Provides better visualization and error handling

---

# **5. Pipeline Components**

### **1. Agent**

Defines where the pipeline will run.

Examples:

```groovy
agent any
agent none
agent { label 'linux' }
agent { docker 'maven:3.9' }
```

### **2. Environment**

Used for environment variables.

```groovy
environment {
    APP_ENV = "dev"
    VERSION = "1.0"
}
```

### **3. Stages**

Logical group of tasks.

```groovy
stages {
    stage('Build') { ... }
    stage('Test') { ... }
}
```

### **4. Steps**

Actual commands inside a stage.

```groovy
steps {
    sh 'mvn clean install'
}
```

### **5. Post Section**

Handles notifications and actions after pipeline execution.

```groovy
post {
    success {
        echo "Pipeline succeeded"
    }
    failure {
        echo "Pipeline failed"
    }
}
```

---

# **6. Advanced Concepts in Jenkins Pipeline**

## **1. Parallel Stages**

Run jobs simultaneously.

```groovy
stage('Parallel Tests') {
    parallel {
        stage('Unit Tests') {
            steps { sh 'mvn test' }
        }
        stage('Integration Tests') {
            steps { sh './run_integration.sh' }
        }
    }
}
```

---

## **2. Input Step**

Pauses the pipeline for manual approval.

```groovy
stage('Approval') {
    steps {
        input "Deploy to production?"
    }
}
```

---

## **3. Using Docker in Pipeline**

```groovy
pipeline {
    agent {
        docker { image 'maven:3.9'; args '-v /tmp:/tmp' }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}
```

---

## **4. Shared Libraries**

Allows you to reuse scripts across multiple pipelines.

* Helps maintain large infrastructures
* Moves reusable logic to a central repo

Example usage:

```groovy
@Library('my-shared-lib') _
pipeline {
    ...
}
```

---

## **5. Credentials Management**

Using credentials store:

```groovy
withCredentials([usernamePassword(credentialsId: 'mycreds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
    sh 'echo $USER'
}
```

---

## **6. Agents and Nodes**

* **Node** = Jenkins worker machine
* **Agent** = Where pipeline or stage executes

You can target specific nodes:

```groovy
agent { label 'ubuntu-node' }
```

---

# **7. Multibranch Pipeline**

Used in enterprises.

### Features:

* Automatically detects branches
* Automatically triggers builds
* Perfect for GitHub/GitLab branching model
* Each branch has its own Jenkinsfile

Used heavily in:

* GitOps workflows
* Microservices
* Feature-branch deployments

---

# **8. Blue Ocean**

A modern UI for Jenkins to visualize pipelines with:

* Graphical pipeline view
* Better logs
* User-friendly design

---

# **9. Common Pipeline Patterns**

### **1. Build → Test → Package → Deploy**

### **2. Build Matrix**

Used for multiple OS / versions.

### **3. Rolling Deployments**

Used with Kubernetes or Ansible.

### **4. Canary Deployments**

Used in modern DevOps pipelines.

---

# **10. Best Practices**

* Always use **Declarative Pipeline** unless complex Groovy logic is needed.
* Store Jenkinsfile in the repository root.
* Use **agent none** and specify agent at stage level when possible.
* Split long pipelines using **Shared Libraries**.
* Use proper **post** blocks for cleanup.
* Use **retry**, **timeout**, and **parallel** cautiously.
* Protect credentials using **withCredentials** block.
* Use meaningful stage names.

---

# **11. Summary**

Jenkins Pipeline is essential for modern CI/CD.
Companies overwhelmingly prefer the **Declarative Pipeline** for its readability, maintainability, and strong validation.
Scripted Pipeline is still valuable for advanced custom logic.
Pipelines enable automated, consistent, and secure delivery processes across environments.
