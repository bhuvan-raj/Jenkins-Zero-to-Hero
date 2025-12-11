# **Lab: Jenkins Email Notification Setup and Usage**

---

## **Objective**

1. Configure Jenkins to send email notifications using Gmail SMTP.
2. Test email notifications.
3. Use email notifications in:

   * Freestyle Jobs
   * Pipeline Jobs
4. Learn to customize email content, subject, and triggers.

---

## **Prerequisites**

1. Jenkins installed and running.
2. Gmail account with **App Password** enabled (2FA must be on).

   * Generate App Password: [Google App Password Guide](https://support.google.com/accounts/answer/185833)
3. Email Extension plugin (`email-ext`) installed in Jenkins.
4. Basic understanding of Freestyle and Pipeline jobs.

---

## **Step 1: Configure SMTP in Jenkins**

1. Go to **Jenkins Dashboard → Manage Jenkins → Configure System**
2. Scroll to **Extended E-mail Notification** (or Email Notification).
3. Configure as follows:

| Field               | Value                             |
| ------------------- | --------------------------------- |
| SMTP server         | `smtp.gmail.com`                  |
| Default user e-mail | `yourmail@gmail.com`              |
| Use SSL             | ❌ (leave unchecked, use STARTTLS) |
| Use TLS             | ✅                                 |
| SMTP Port           | `587`                             |
| SMTP Authentication | ✅ Enable                          |
| Username            | `yourmail@gmail.com`              |
| Password            | `your_app_password_here`          |
| Reply-To Address    | Optional                          |
| Charset             | `UTF-8`                           |

4. Click **Test configuration by sending test email**.

   * Enter **recipient**: `yourmail@gmail.com`
   * Click **Test**
   * Verify that you receive an email.

> **Tip:** Make sure there are **no spaces** in the app password.

---

## **Step 2: Freestyle Job Email Notification**

### **Step 2.1: Create a Freestyle Job**

1. Go to **Jenkins Dashboard → New Item → Freestyle project**
2. Name it `Freestyle_Email_Test`
3. Click **OK**

### **Step 2.2: Add Build Step**

1. Add a **Build Step → Execute shell / Windows batch command**
2. Add:

```bash
echo "This is a test build"
exit 1   # Optional: forces failure to trigger email
```

### **Step 2.3: Add Post-build Email Notification**

1. Scroll to **Post-build Actions**

2. Click **Add post-build action → Editable Email Notification**

3. Configure:

   * **Project Recipient List:** `yourmail@gmail.com`
   * **Triggers:** select **Failure** (or Success/Always)
   * **Subject:** `$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS`
   * **Content:**

     ```
     Build Status: $BUILD_STATUS
     Console Output: $BUILD_URL
     ```

4. Save and **Build Now**

5. Check your email for notification.

---

## **Step 3: Pipeline Job Email Notification**

### **Step 3.1: Create a Pipeline Job**

1. Go to **New Item → Pipeline**
2. Name it `Pipeline_Email_Test`
3. Click **OK**

### **Step 3.2: Configure Pipeline (Jenkinsfile)**

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo "Running Pipeline Build..."
                // Uncomment below to simulate failure
                // sh 'exit 1'
            }
        }
    }

    post {
        success {
            emailext(
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Pipeline Build Succeeded.\nCheck console: ${env.BUILD_URL}",
                to: "yourmail@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Pipeline Build Failed.\nCheck console: ${env.BUILD_URL}",
                to: "yourmail@gmail.com"
            )
        }
    }
}
```

4. Save and **Build Now**
5. Verify email notifications for **Success** or **Failure**

---

## **Step 4: Advanced Email Features (Optional)**

1. **HTML Emails**

```groovy
emailext(
    mimeType: 'text/html',
    subject: "Build Notification",
    body: """
        <h2>Build Report</h2>
        <p>Job: ${env.JOB_NAME}</p>
        <p>Build: ${env.BUILD_NUMBER}</p>
        <p>Status: ${currentBuild.currentResult}</p>
        <a href="${env.BUILD_URL}">View Console Output</a>
    """,
    to: "yourmail@gmail.com"
)
```

2. **Attachments**: Use `attachmentsPattern: 'reports/*.txt'` to attach files.

3. **Multi-recipient emails**:

```
to: "team1@example.com, team2@example.com"
```

---

## **Step 5: Verification**

1. Trigger builds manually to verify emails.
2. Check **success/failure triggers** work as expected.
3. Adjust **email content** and **triggers** based on project needs.

---
