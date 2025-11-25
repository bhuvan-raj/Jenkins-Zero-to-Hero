
# **Jenkins Architecture – In-Depth Study Notes**
<img src="https://github.com/bhuvan-raj/Jenkins-Zero-to-Hero/blob/main/assets/jenkins.png" alt="Banner" />


## **1. Overview of Jenkins**

Jenkins is an open-source automation server primarily used for continuous integration (CI) and continuous delivery (CD). Its architecture is designed to be modular, distributed, and highly extensible, enabling the automation of build, test, and deployment pipelines across numerous environments.

---

# **2. High-Level Architecture**

At a broad level, Jenkins architecture consists of:

### **2.1 Jenkins Master (Controller)**

This is the central brain of Jenkins. It manages:

* The UI and REST API
* Job configuration and orchestration
* Queue management
* Dispatching build tasks to agents
* Plugin management
* Storing build logs, job history, credentials, and global configurations

In recent terminology, “Master” has been renamed to **Controller**.

### **2.2 Jenkins Agent (Node)**

A Jenkins agent is a machine—physical, VM, Docker container, or Kubernetes pod—that performs the actual job execution.
Agents are used to:

* Offload workloads
* Scale horizontally
* Run platform-specific builds (Linux/Windows/Mac)
* Run parallel jobs

Agents communicate with the controller via:

* SSH
* JNLP/WebSockets
* Docker plugins
* Kubernetes agent plugins

---

# **3. Jenkins Internal Components (Detailed Breakdown)**

## **3.1 Jenkins Controller Components**

### **3.1.1 Web UI & CLI Layer**

* Provides dashboards, job views, logs, and real-time console outputs.
* Jenkins CLI allows admin operations (create jobs, trigger builds, manage nodes) via commands.

### **3.1.2 Job/Project Configurations**

Stored as XML files in the Jenkins home directory. Types include:

* Freestyle jobs
* Pipeline jobs (Declarative & Scripted)
* Multibranch pipelines
* GitHub Organization jobs

### **3.1.3 Build Queue**

When a build is triggered, it enters the **queue**.
The controller checks:

* Whether agents have available executors
* Node labels
* Resource availability
* Job restrictions

Only after these conditions are satisfied, the controller assigns the job to an executor.

### **3.1.4 Executors (Core Concept)**

An **executor** is a slot in which Jenkins runs a build.
Executors belong either to:

* The controller (default: 2 executors)
* A connected agent (you configure how many executors an agent can run)

#### **How Executors Work**

* A build can start only if an executor is free.
* More executors mean more parallel builds.
* Each executor runs **one job at a time**.
* Executors consume CPU/RAM of the underlying machine.

#### **Why Executors Matter**

Executors determine:

* System capacity
* Job throughput
* Parallelism level
* How distributed builds are handled

#### **Best Practices**

* Set controller executors = 0 (controller should not execute builds; only orchestrate)
* Size agent executors based on CPU/RAM
* Avoid over-allocating executors to prevent resource exhaustion

---

## **3.2 Agents and Node Components**

### **3.2.1 Node Labels**

Labels help target specific agents. Examples:

* `linux`
* `docker`
* `java17`
* `gpu`

Jobs can specify label requirements to run on appropriate nodes.

### **3.2.2 Agent Launch Mechanisms**

Different approaches used depending on environment:

* **SSH agents** – common for Linux servers
* **JNLP agents** – useful for cloud, Windows, firewalled environments
* **Docker slaves plugin** – temporary agents via Docker
* **Kubernetes plugin** – dynamic pod-based agents (most scalable)

### **3.2.3 Workspace**

Each build gets its own **workspace** directory on the agent:

* Contains source code checkout
* Build artifacts
* Temporary files

After the build, workspace may be cleaned or archived depending on configuration.

### **3.2.4 Tools Configuration**

Jenkins manages:

* JDK versions
* Maven, Gradle, Python, NodeJS
* Custom tool installations

Agents use these tools during build execution.

---

# **4. Plugins – The Heart of Jenkins Architecture**

Plugins extend Jenkins functionality. Jenkins has over 1,800 plugins.

### **4.1 Types of Critical Plugins**

* **SCM Plugins:** Git, GitHub, Bitbucket
* **Build Tools:** Maven, Gradle, Docker
* **Credentials & Secrets:** Credentials Binding plugin
* **User & Access Control:** LDAP, SSO, Matrix Authorization
* **Cloud & Agent Management:** Kubernetes, EC2, Docker agents
* **Visualization:** Blue Ocean, Pipeline Steps, Job views

### **4.2 Plugin Dependency Architecture**

Plugins rely on one another, creating dependency trees.
The Jenkins controller checks:

* Compatibility
* Required dependencies
* Update levels

A broken plugin can affect the entire system.

---

# **5. Build Pipeline Architecture**

### **5.1 Pipeline (Jenkinsfile)**

Pipeline as code is executed through:

* **Declarative Pipeline** (structured, recommended)
* **Scripted Pipeline** (Groovy-based, more flexible)

Pipelines run inside executors on agents.

### **5.2 Stages & Steps**

* **Stages:** Logical blocks (Build, Test, Deploy)
* **Steps:** Actual commands executed

Pipeline architecture supports:

* Parallel stages
* Conditional flows
* Shared libraries

---

# **6. Distributed Build Architecture**

## **6.1 When to Use Distributed Builds**

* Heavy builds
* Multi-platform testing
* Isolation requirements
* Large engineering teams

### **6.2 Controller-Agent Communication**

Uses different protocols:

1. **HTTP/HTTPS** – UI, plugins
2. **SSH/JNLP/WebSocket** – agent connection
3. **File I/O** – storing logs, artifacts

### **6.3 Scalability**

Jenkins scales horizontally by adding more agents or dynamically provisioning them via cloud integrations.

---

# **7. Jenkins Storage Architecture**

### **7.1 Jenkins Home Directory**

Includes:

* Job configurations (XML)
* Build logs
* Plugin data
* Credentials store
* System configurations

### **7.2 Artifacts Storage**

Artifacts can be:

* Stored locally
* Pushed to artifact repositories (Nexus, Artifactory, GitHub Packages)

### **7.3 Log Storage**

Stored under each job’s directory.
Plugins can ship logs externally (e.g., CloudWatch, ElasticSearch).

---

# **8. Security Architecture**

### **8.1 Authentication**

Methods:

* Jenkins internal users
* LDAP/Active Directory
* SSO (SAML/OIDC)
* GitHub OAuth

### **8.2 Authorization**

Access models:

* Full access
* Project-based matrix
* Role-based access control (via plugins)

### **8.3 Credentials Store**

Sensitive data encrypted with master key:

* SSH keys
* AWS keys
* Tokens
* Secrets

Accessed in pipelines through credentials binding.

---

# **9. Lifecycle of a Jenkins Build**

1. User commits code → SCM webhook triggers Jenkins
2. Build enters **queue**
3. Controller checks available executors
4. A matching agent with a free executor is selected
5. Workspace is prepared
6. Build/pipeline steps are executed
7. Logs streamed to UI
8. Artifacts archived
9. Post-build actions executed (email, deploy, notify)
10. Executor becomes free for next job

---

# **10. Quick Summary for Exams or Teaching**

✔ **Controller:** Orchestrates jobs, stores configs, manages queue
✔ **Agents:** Execute builds
✔ **Executors:** Run one job at a time; determine parallelism
✔ **Plugins:** Extend Jenkins functionality
✔ **Pipeline:** Automates CI/CD via Jenkinsfile
✔ **Distributed Architecture:** Multiple agents for scalability
✔ **Security:** Authentication, authorization, credential management

---
