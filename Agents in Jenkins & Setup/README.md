# **Jenkins Agents – In-Depth Study Notes**

Jenkins agents (previously called “slaves”) are machines or environments that perform the **actual job execution**. The controller (master) delegates build tasks to agents. Agents allow Jenkins to **scale horizontally**, run jobs on different platforms, and isolate workloads.

---

# **1. Overview of Jenkins Agents**

* Agents are machines, containers, or pods connected to the Jenkins controller.
* They can execute **one or more executors**.
* Agents improve **scalability**, **resource utilization**, and **parallel execution**.

---

# **2. Two Main Types of Agents**

At a conceptual level, all Jenkins agents fall under two categories:

## **2.1 Static Agents (Permanent Agents)**

* Also called **dedicated agents**.
* Configured manually and **always available** to the controller.
* Typical setup:

  * Physical servers
  * Virtual machines
  * Dedicated nodes in cloud environments

### **Characteristics**

* Permanently connected to the controller (unless offline manually).
* Assigned a fixed number of **executors**.
* Useful for **consistent environments** (e.g., testing specific OS, pre-installed tools).
* Managed via **Manage Jenkins → Nodes**.



---

## **2.2 Dynamic Agents (On-Demand Agents)**

* Also called **ephemeral or cloud agents**.
* Provisioned **automatically when a build is triggered** and destroyed after completion.
* Useful for **scalable pipelines**, reducing idle resources.

### **Characteristics**

* Created automatically by Jenkins plugins or scripts.
* Can run on various platforms: cloud VMs, Docker containers, Kubernetes pods.
* Highly scalable – ideal for large CI/CD pipelines.
* Avoid resource wastage by destroying agents after builds.

---

# **3. Specific Agent Types (All Derived from Static or Dynamic Concepts)**

### **3.1 Docker Agents**

* Can be static (permanent Docker container) or dynamic (ephemeral container for each build).
* Useful for **isolated build environments**.
* Plugins: **Docker plugin**, **Docker Pipeline plugin**.

**Workflow Example**:

* Jenkins pulls a Docker image with required tools
* Starts a container as an agent
* Executes the pipeline
* Container is removed after completion (dynamic setup)

---

### **3.2 Kubernetes Agents**

* Agents run as **Kubernetes pods** (ephemeral).
* Jenkins dynamically creates pods for each pipeline build.
* Plugins: **Kubernetes Plugin**.

**Features**:

* Each stage of a pipeline can run on a separate pod.
* Ideal for **scalable, cloud-native CI/CD**.
* Fully supports **parallel pipelines** without worrying about static infrastructure.

---

### **3.3 Cloud Agents (AWS, Azure, GCP)**

* Dynamic agents provisioned on cloud VMs.
* Plugins: EC2, Azure VM Agents, Google Cloud plugin.
* **Use case**: Auto-scale build agents on cloud infrastructure.

**Workflow**:

1. Jenkins controller requests a cloud VM
2. Agent is provisioned with pre-installed tools or container images
3. Executes build and shuts down VM to save costs

---

### **3.4 Other Types**

* **Windows Agents**: Usually static, connected via JNLP or Windows Service.
* **Bare Metal / Physical Agents**: Dedicated servers for heavy builds or specific hardware requirements.
* **External Agents**: Non-Jenkins-managed systems monitored externally.

---

# **4. Comparison: Static vs Dynamic Agents**

| Feature              | Static Agent                  | Dynamic Agent                    |
| -------------------- | ----------------------------- | -------------------------------- |
| Availability         | Always online                 | Created on-demand                |
| Provisioning         | Manual                        | Automated via plugin or script   |
| Scalability          | Limited by fixed resources    | Highly scalable                  |
| Resource Utilization | Might be idle sometimes       | Efficient – destroys after use   |
| Setup Complexity     | Moderate                      | Requires plugin & cloud config   |
| Use Case             | Dedicated builds, specific OS | CI/CD pipelines, parallel builds |

---

# **5. Key Takeaways**

1. **All agent types (Docker, Kubernetes, cloud VMs) are essentially either static or dynamic**.

   * Docker or Kubernetes agents are usually **dynamic/ephemeral**.
   * Dedicated Windows/Linux machines are usually **static**.
2. Agents can have multiple **executors**, enabling parallel builds.
3. Labels are critical for **job-to-agent mapping**.
4. Dynamic agents are ideal for **scalable, cloud-native CI/CD**, while static agents are suited for **stable, consistent environments**.

---

# **Jenkins EC2 Agents – Setup Guide**

Jenkins can use EC2 instances as agents (nodes) to offload builds. EC2 agents can be **static** (always running) or **dynamic** (provisioned on-demand).

---

## **Prerequisites**

1. **Jenkins Controller Setup**

   * Installed on a server accessible by the EC2 instances.
   * Admin access to Jenkins.
   * Plugins:

     * **Amazon EC2 Plugin**
     * **SSH Slaves Plugin** (optional, for SSH launch)
     * **Pipeline Plugin** (optional, if pipelines will use EC2 agents)
2. **AWS Account**

   * IAM user with access to:

     * EC2: `DescribeInstances`, `RunInstances`, `TerminateInstances`
     * VPC, Subnets, Security Groups
   * Access Key ID & Secret Key
3. **EC2 Requirements**

   * AMI with OS Jenkins supports (Ubuntu, Amazon Linux, Windows)
   * Security group allows SSH (Linux) or RDP (Windows) access from Jenkins controller
   * Optional: Pre-install JDK, Git, Maven, Docker, etc.

---

# 1. Setting Up a Static EC2 Agent

Static EC2 agents are **permanent Jenkins nodes** that always stay online unless manually stopped.

---

## **Step 1: Launch an EC2 Instance**

1. Go to **AWS Console → EC2 → Launch Instances**
2. Select:

   * **AMI**: Ubuntu 22.04 LTS (example)
   * **Instance type**: t2.medium (adjust for builds)
   * **Key pair**: Create or select an existing key pair
   * **Security group**: Allow **SSH** (port 22) from Jenkins controller IP
3. Launch instance and note **public/private IP**

---

## **Step 2: Prepare the EC2 Instance**

1. SSH into EC2:

   ```bash
   ssh -i your-key.pem ubuntu@ec2-public-ip
   ```
2. Install required software:

   ```bash
   sudo apt update
   sudo apt install -y openjdk-17-jdk git maven docker.io
   ```
3. Ensure SSH works from Jenkins controller.

---

## **Step 3: Add Static Agent in Jenkins**

1. Go to **Jenkins Dashboard → Manage Jenkins → Manage Nodes and Clouds → New Node**
2. Enter:

   * **Node Name**: `Static-EC2-Agent`
   * **Type**: Permanent Agent
3. Configure:

   * **Remote root directory**: `/home/ubuntu/jenkins` (workspace)
   * **Number of Executors**: `2` (adjust based on CPU)
   * **Labels**: `linux ec2 static`
   * **Launch Method**: `Launch agents via SSH`

     * **Host**: EC2 private/public IP
     * **Credentials**: Add SSH key credentials (Jenkins → Credentials → Add → SSH Key)
     * **Host Key Verification Strategy**: Default or Manually trusted
   * **Availability**: Always
4. Save and test connection → Agent should go online.

---

## **Step 4: Test the Static Agent**

* Go to **Manage Jenkins → Nodes → Static-EC2-Agent → Build a test job**
* Ensure builds run successfully on this node.

---

# **2. Setting Up a Dynamic EC2 Agent**

Dynamic EC2 agents are **created and destroyed on-demand** for builds, ideal for scalable CI/CD.

---

## **Step 1: Install the Amazon EC2 Plugin**

1. Go to **Manage Jenkins → Manage Plugins → Available**
2. Search for `Amazon EC2 Plugin`
3. Install and restart Jenkins if needed.

---

## **Step 2: Add AWS Credentials in Jenkins**

1. Go to **Manage Jenkins → Credentials → System → Global credentials**
2. Add:

   * **Kind**: AWS Credentials
   * **Access Key ID & Secret Key**
   * **ID**: `aws-ec2-credentials`

---

## **Step 3: Configure EC2 Cloud in Jenkins**

1. Go to **Manage Jenkins → Manage Nodes and Clouds → Configure Clouds → Add a new Cloud → Amazon EC2**
2. Configure Cloud settings:

   * **Name**: `EC2-Dynamic-Cloud`
   * **Region**: e.g., `us-east-1`
   * **Credentials**: Select `aws-ec2-credentials`
   * **Jenkins URL**: Default or controller URL
   * **Instance Cap**: Max number of instances Jenkins can spin up
   * Optional: Use custom AMI ID if pre-installed tools are required

---

## **Step 4: Configure EC2 Agent Template**

1. Click **Add Template**
2. Configure template:

   * **AMI ID**: e.g., `ami-0abcdef1234567890`
   * **Instance Type**: t2.medium
   * **Remote root directory**: `/home/ubuntu/jenkins`
   * **Labels**: `linux ec2 dynamic`
   * **Launch Method**: SSH

     * SSH Key credentials
     * Username: `ubuntu` (or OS-specific)
   * **Usage**: Use this node as much as possible
   * **Idle Termination**: `5 minutes` (agent is terminated if idle for this time)

---

## **Step 5: Configure Jobs to Use Dynamic Agents**

* In the job configuration or pipeline:

  * Freestyle: Set **Restrict where this project can be run** → `Label Expression`: `linux ec2 dynamic`
  * Pipeline: Use `agent { label 'linux ec2 dynamic' }`

**Example Declarative Pipeline**:

```groovy
pipeline {
    agent { label 'linux ec2 dynamic' }
    stages {
        stage('Build') {
            steps {
                sh 'echo "Building on dynamic EC2 agent"'
                sh 'mvn clean install'
            }
        }
    }
}
```

---

## **Step 6: Test Dynamic Agent**

1. Trigger a build
2. Jenkins will:

   * Launch EC2 instance dynamically using template
   * Run the build in the assigned executor
   * Terminate the instance after idle timeout

---

# **3. Best Practices**

| Recommendation                                        | Reason                                    |
| ----------------------------------------------------- | ----------------------------------------- |
| Use **static agents** for heavy or specialized builds | Reduce EC2 spin-up time                   |
| Use **dynamic agents** for CI/CD pipelines            | Cost-efficient, scalable                  |
| Label agents properly                                 | Helps jobs select the correct environment |
| Configure **idle termination** for dynamic agents     | Save cloud costs                          |
| Use pre-baked AMIs with necessary tools               | Faster agent provisioning                 |
| Limit instance cap in EC2 cloud config                | Avoid unexpected AWS charges              |

---

# **4. Summary**

| Feature       | Static EC2 Agent           | Dynamic EC2 Agent                   |
| ------------- | -------------------------- | ----------------------------------- |
| Availability  | Always online              | On-demand                           |
| Provisioning  | Manual EC2 instance        | Jenkins auto-launch from template   |
| Cost          | Continuous running cost    | Pay-per-use, terminated after build |
| Scalability   | Limited by fixed instances | Highly scalable                     |
| Best Use Case | Long-running, heavy builds | CI/CD pipelines, ephemeral jobs     |
| Plugins Used  | SSH Slaves Plugin          | Amazon EC2 Plugin                   |

---


