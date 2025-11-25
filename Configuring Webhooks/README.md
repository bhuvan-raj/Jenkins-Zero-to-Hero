# Webhooks for Github
<img src="https://github.com/bhuvan-raj/Jenkins-Zero-to-Hero/blob/main/assets/webhook.jpg" alt="Banner" />

## **1. What is a Webhook?**

A **webhook** is a **user-defined HTTP callback** that allows one system to notify another system automatically when a certain event occurs.

* In the context of **GitHub and Jenkins**:

  * GitHub sends an HTTP POST request to a configured URL in Jenkins whenever certain events occur (push, pull request, tag creation, etc.).
  * Jenkins receives the payload and can **trigger jobs automatically** without polling the repository.

**Advantages of Webhooks:**

* Immediate triggering of builds
* Reduces load from polling
* Ensures CI/CD pipelines react instantly to changes

---

## **2. Prerequisites**

1. **Jenkins**

   * Running and accessible from GitHub (public URL or tunneled with ngrok)
   * Admin privileges

2. **Required Plugins**

   * **GitHub Plugin**
   * **GitHub Branch Source Plugin** (for pipelines/multibranch)
   * Optional: **Pipeline Plugin** for Jenkinsfile-based pipelines

3. **GitHub Repository**

   * Admin access to configure webhooks
   * Repository URL (public or private)

4. **Jenkins Credentials**

   * GitHub personal access token for private repos
   * Stored securely in Jenkins credentials

---

## **3. Configure Jenkins Job for Webhooks**

### **3.1 Freestyle Project**

1. Create a **New Item → Freestyle Project**
2. Under **Source Code Management → Git**:

   * Repository URL
   * Credentials (if private)
   * Branch to build
3. Under **Build Triggers**:

   * Check **GitHub hook trigger for GITScm polling**

### **3.2 Pipeline Project**

1. Create **New Item → Pipeline → OK**
2. Configure **Pipeline script from SCM**:

   * SCM: Git
   * Repository URL
   * Branch
   * Credentials if private
3. Under **Build Triggers**:

   * Check **GitHub hook trigger for GITScm polling**

> This enables Jenkins to listen for webhook events from GitHub instead of polling.

---

## **4. Configure GitHub Webhook**

1. Go to **GitHub repository → Settings → Webhooks → Add webhook**
2. Configure:

   * **Payload URL**: `http://<JENKINS_URL>/github-webhook/`

     * Example: `http://jenkins.example.com/github-webhook/`
   * **Content type**: `application/json`
   * **Secret**: Optional security token (recommended)
   * **Which events?**:

     * Choose **Just the push event** (typical)
     * Or **Send me everything** for all events
3. Click **Add webhook**

---

## **5. Add GitHub Credentials in Jenkins (for private repo)**

1. Go to **Manage Jenkins → Credentials → System → Global credentials → Add Credentials**
2. Add:

   * **Kind**: Secret Text (GitHub token) or Username/Password
   * **ID**: Optional (e.g., `github-token`)
   * **Description**: `GitHub Personal Access Token for webhook`
3. Use these credentials in the Jenkins job SCM configuration

---

## **6. Test the Webhook**

1. Push a commit to the repository
2. Jenkins should automatically trigger a build
3. Check:

   * Jenkins **Console Output** for build logs
   * GitHub **Webhook → Recent Deliveries** to see POST status (HTTP 200 indicates success)

---

## **7. Optional Security Enhancements**

* **Secret Token**:

  * Set in GitHub webhook
  * Validate in Jenkins using plugins
* **IP Whitelisting**: Allow only GitHub IPs
* **HTTPS Payload URL**: Encrypt data in transit
* **Restrict triggers to specific branches**: Avoid unnecessary builds

---

## **8. Notes**

* For **local Jenkins instances**, you may need **ngrok** or similar to expose Jenkins to GitHub.
* Webhooks are preferred over **polling** because they are:

  * Faster (instant trigger)
  * Efficient (reduce resource usage)
* Multiple Jenkins jobs can use a single webhook; Jenkins filters events based on repository and branch.

---

## **9. Summary of Steps**

1. Install required Jenkins plugins
2. Create Jenkins job (Freestyle or Pipeline)
3. Enable **GitHub hook trigger for GITScm polling**
4. Add **GitHub webhook** in repository settings:

   * URL: `http://<JENKINS_URL>/github-webhook/`
   * Event: push (or others)
5. Configure credentials if repo is private
6. Push code and verify build triggers automatically

---
