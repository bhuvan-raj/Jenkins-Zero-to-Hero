# Jenkins RBAC (Role-Based Access Control)

This guide explains how to implement Role-Based Access Control in Jenkins using the **Role-based Authorization Strategy** plugin, and includes hands-on lab steps to set it up from scratch.

---

## Table of Contents

1. [Overview](#overview)
2. [Why RBAC in Jenkins](#why-rbac-in-jenkins)
3. [Key Concepts](#key-concepts)
4. [Prerequisites](#prerequisites)
5. [Lab Steps](#lab-steps)
6. [Role Types Explained](#role-types-explained)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)
9. [References](#references)

---

## Overview

By default, Jenkins uses simple security models (e.g., "logged-in users can do anything" or matrix-based security applied globally). For real-world, multi-team environments, this is too coarse. **RBAC** lets you define fine-grained permissions based on:

- **Who** the user is (or what group they belong to)
- **What** they can do (build, configure, delete, view, etc.)
- **Where** they can do it (globally, per-project, per-folder/node)

Jenkins achieves this via the **Role-based Authorization Strategy** plugin (also referred to as Role Strategy Plugin).

---

## Why RBAC in Jenkins

- Prevent accidental or malicious changes to production pipelines
- Give developers access only to their own jobs/folders
- Allow QA/ops teams read-only or execute-only access
- Satisfy audit and compliance requirements
- Reduce blast radius of compromised credentials

---

## Key Concepts

| Term | Description |
|---|---|
| **Global Roles** | Apply instance-wide (e.g., Admin, Anonymous, Read-only User) |
| **Item Roles** | Apply to specific jobs/folders matched by a regex pattern |
| **Agent/Node Roles** | Apply to specific build agents/nodes |
| **Permissions** | Granular actions like Job/Build, Job/Configure, Job/Delete, Overall/Administer |
| **Assignments** | Mapping of users/groups to roles |

---

## Prerequisites

- A running Jenkins instance (2.4xx LTS or later recommended)
- Admin access to Jenkins
- At least 2–3 test user accounts (or LDAP/AD group if testing enterprise auth)
- Sample jobs/folders to test scoped permissions

---

## Lab Steps

### Lab 1: Install the Role-based Authorization Strategy Plugin

1. Go to **Manage Jenkins → Plugins → Available plugins**.
2. Search for **Role-based Authorization Strategy**.
3. Select it and click **Install without restart**.
4. Once installed, go to **Manage Jenkins → Security**.

### Lab 2: Enable Role-Based Strategy

1. Under **Manage Jenkins → Security**, scroll to **Authorization**.
2. Select **Role-Based Strategy**.
3. Click **Save**.

> ⚠️ At this point, no roles exist yet — access may be restricted. Log in as admin (the initial admin account retains full access).

### Lab 3: Create Test Users

1. Go to **Manage Jenkins → Users → Create User**.
2. Create three users:
   - `dev_user`
   - `qa_user`
   - `ops_admin`

### Lab 4: Configure Global Roles

1. Go to **Manage Jenkins → Manage and Assign Roles → Manage Roles**.
2. Under **Global roles**, add new roles:
   - `readonly` → check `Overall/Read`
   - `developer` → check `Overall/Read`, `Job/Build`, `Job/Read`, `Job/Workspace`
   - `admin` → check all permissions (or use existing `admin` role)
3. Click **Save**.

### Lab 5: Create Item (Project) Roles

1. Still on **Manage Roles**, scroll to **Item roles**.
2. Add a role named `team-a-jobs`.
3. Set the **Pattern** field to match job names, e.g.:
   ```
   team-a-.*
   ```
4. Assign permissions: `Job/Build`, `Job/Configure`, `Job/Read`, `Job/Workspace`.
5. Click **Save**.

### Lab 6: Assign Roles to Users

1. Go to **Manage Jenkins → Manage and Assign Roles → Assign Roles**.
2. Under **Global roles**, assign:
   - `dev_user` → `developer`
   - `qa_user` → `readonly`
   - `ops_admin` → `admin`
3. Under **Item roles**, assign:
   - `dev_user` → `team-a-jobs`
4. Click **Save**.

### Lab 7: Create Test Jobs

1. Create two Freestyle or Pipeline jobs:
   - `team-a-build`
   - `team-b-build`
2. Confirm the naming matches the regex pattern used in Lab 5 (`team-a-.*`).

### Lab 8: Validate Access Control

Log in as each user (use incognito windows or separate browsers) and verify:

| User | Expected Behavior |
|---|---|
| `dev_user` | Can view/build `team-a-build`; **cannot** see or access `team-b-build` |
| `qa_user` | Can view all jobs (read-only); **cannot** trigger builds or edit configs |
| `ops_admin` | Full access to everything |

### Lab 9 (Optional): Folder-Based RBAC

1. Install the **Folders** plugin if not already installed.
2. Create a folder `team-a/` and move `team-a-build` into it.
3. In **Manage Roles → Item roles**, update pattern to:
   ```
   team-a/.*
   ```
4. Re-test access as `dev_user`.

### Lab 10 (Optional): Integrate with LDAP/AD Groups

1. Go to **Manage Jenkins → Security → Security Realm**.
2. Configure **LDAP** with your directory server details.
3. In **Assign Roles**, add a **group** (prefix with `@` or per plugin convention, e.g., `@qa-team`) instead of individual users.
4. Validate that group membership grants the expected role.

---

## Role Types Explained

| Role Type | Scope | Example Use Case |
|---|---|---|
| Global Role | Entire Jenkins instance | Admins, read-only auditors |
| Item Role | Jobs/folders matching a regex | Team-specific job access |
| Agent Role | Specific build nodes/agents | Restrict who can use a GPU/prod agent |

---

## Best Practices

- **Principle of least privilege** — grant only what's needed.
- Use **folders + item roles** together for scalable multi-team setups.
- Avoid assigning roles to individual users when LDAP/AD groups are available — easier to manage at scale.
- Regularly audit **Assign Roles** page for stale users/permissions.
- Combine RBAC with **Jenkins audit trail plugin** for traceability.
- Avoid overly broad regex patterns (e.g., `.*`) in item roles — this defeats the purpose of scoping.
- Back up `config.xml` files before making authorization changes, in case you get locked out.

---

## Troubleshooting

| Issue | Fix |
|---|---|
| Locked out after enabling Role Strategy | Edit `$JENKINS_HOME/config.xml` directly (via file system access) and reset `authorizationStrategy` to `hudson.security.FullControlOnceLoggedInAuthorizationStrategy`, restart Jenkins |
| Item role not applying | Double-check the regex pattern — it must match the **full job path**, not just partial names |
| User has no access despite role assignment | Confirm the plugin is enabled under **Authorization**, not just installed |
| Group-based roles not working | Ensure your security realm (LDAP/AD) is correctly configured and group names match exactly |

---

## References

- [Role-based Authorization Strategy Plugin (Jenkins Wiki)](https://plugins.jenkins.io/role-strategy/)
- [Jenkins Security Documentation](https://www.jenkins.io/doc/book/security/)
- [Jenkins Folders Plugin](https://plugins.jenkins.io/cloudbees-folder/)

---

## License

This README is provided as-is for internal training/lab purposes.
