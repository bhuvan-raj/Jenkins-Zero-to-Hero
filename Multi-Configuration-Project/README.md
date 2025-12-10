
# **Multi-Configuration Project in Jenkins (Matrix Project)**

A **Multi-Configuration Project**, also known as a **Matrix Project**, is a job type in Jenkins designed to run the **same build** across **multiple configurations**, such as:

* Different environments
* Different parameter values
* Different operating systems
* Different JDK or toolchain versions
* Different build targets (e.g., multiple Docker registries)
* Any defined combination of variables

This feature makes it especially useful in testing, cross-platform validation, and multi-target deployment pipelines.

---

# **1. Purpose of Multi-Configuration Projects**

Multi-Configuration Projects are ideal when:

1. The same build must run under **many combinations** of settings.
2. You want Jenkins to **automatically generate parallel jobs** based on configuration axes.
3. You want a **GUI-driven** alternative to complex Pipeline logic.
4. You need **repeatable matrix builds** without writing a Jenkinsfile.

Common use cases:

* Testing Python app on Python 3.8, 3.9, 3.10
* Building artifacts on Linux + Windows
* Running the same job with different environment variables
* Deploying Docker images to multiple registries
* Running jobs across multiple nodes or labels

---

# **2. Key Concepts**

## **2.1 Axes**

An *axis* defines the variable values the matrix should iterate through.
Example:

| Axis Name | Values                |
| --------- | --------------------- |
| JDK       | jdk8, jdk11           |
| OS        | ubuntu, windows       |
| REGISTRY  | dockerhub, ghpackages |

Jenkins will generate builds for every combination of axis values.

---

## **2.2 Matrix Execution**

Jenkins expands the matrix. Example:

If you define:

```
REGISTRY = [dockerhub, ghpackages]
MODE = [dev, prod]
```

You get **4 parallel builds**:

| REGISTRY   | MODE |
| ---------- | ---- |
| dockerhub  | dev  |
| dockerhub  | prod |
| ghpackages | dev  |
| ghpackages | prod |

Each run executes the same job logic, but with different variables.

---

## **2.3 Axis Types**

1. **User-defined Axis**
   Define custom variable names and values.
   Example: `REGISTRY = dockerhub ghpackages`

2. **Label Axis**
   Runs builds on different Jenkins nodes based on label (e.g., linux, windows).

3. **JDK Axis**
   Runs builds using different installed JDKs.

4. **Slaves Axis** *(deprecated naming; now “nodes”)*
   Force jobs to run on specific nodes.

---

# **3. Job Structure**

A Multi-Configuration job includes:

### **A. Configuration Matrix section**

Where you add axes.

### **B. Build Environment**

Where you specify environments and credential bindings.

### **C. Build Steps**

Where you add shell scripts, Windows commands, Maven/Gradle builds, Docker builds, etc.

### **D. Post-Build Actions**

For artifact archiving, reporting, notifications, etc.

This structure is identical to a Freestyle job, but with added matrix capabilities.

---

# **4. How the Execution Works**

1. Jenkins evaluates all axes.
2. Jenkins generates separate child builds (“matrix runs”).
3. Each child build gets the axis variables injected into its environment.
4. All builds run **in parallel** (unless throttled).
5. Results are aggregated into one summary view.

---

# **5. Common Use Cases**

### **5.1 Cross-platform testing**

Build and test software on multiple OSes:

* Linux
* Windows
* macOS

### **5.2 Toolchain matrix**

Test with different JDK, Node, Python, or Go versions.

### **5.3 Environment matrix**

Run tests under:

* Dev
* Stage
* Prod

### **5.4 Multi-registry Docker builds**

Example:

* `REGISTRY = dockerhub, ghpackages`

Perfect for multi-image publishing labs.

### **5.5 Multi-node execution**

Run the same job on multiple Jenkins nodes for consistency.

---

# **6. Advantages of Multi-Configuration Projects**

| Benefits                  | Description                                                  |
| ------------------------- | ------------------------------------------------------------ |
| **Parallel execution**    | Automatically runs builds simultaneously for faster testing. |
| **No scripting required** | Fully GUI-based; no Jenkinsfile needed.                      |
| **Visual matrix view**    | Jenkins shows matrix results in a grid view.                 |
| **Flexible**              | Supports axis combinations of almost anything.               |
| **Ideal for testing**     | Excellent for QA teams validating across platforms.          |

---

# **7. Limitations**

| Limitation                       | Description                                          |
| -------------------------------- | ---------------------------------------------------- |
| **Not code-based**               | Harder to version control (compared to Jenkinsfile). |
| **Less flexible than Pipelines** | Limited for complex workflows.                       |
| **UI-heavy**                     | Harder to migrate between Jenkins instances.         |
| **Not ideal for modern CI/CD**   | Pipelines are now industry standard.                 |

Still highly useful for teaching, labs, and parallel deployments.

---

# **8. When Not to Use Multi-Configuration Projects**

Avoid matrix jobs when:

* You need advanced CI/CD workflows
* You require stages, conditions, or scripted logic
* You are using GitOps
* You want complete config-as-code portability
* You want to fully automate deployment pipelines

For production, a Declarative Pipeline with `matrix {}` is recommended.

---

# **9. Comparison: Multi-Configuration vs Pipeline Matrix**

| Feature             | Multi-Configuration Project        | Pipeline Matrix           |
| ------------------- | ---------------------------------- | ------------------------- |
| Setup               | GUI-based                          | Code-based (Jenkinsfile)  |
| Portability         | Low                                | High                      |
| Flexibility         | Medium                             | Very High                 |
| Use Case            | Simple matrix jobs, teaching, labs | Real CI/CD systems        |
| Parallelism         | Yes                                | Yes                       |
| Credential handling | Manual                             | Automated via Jenkinsfile |

---

# **10. Example Use Case: Multi-Registry Docker Deployment**

Define axis:

```
REGISTRY: dockerhub ghpackages
```

Child runs:

* Build & push to Docker Hub
* Build & push to GitHub Packages

Matrix jobs allow these to run in parallel with no scripting logic needed in Jenkinsfile. Only a simple shell script is required.

---

# **11. Visual Summary Diagram**

```
            ┌─────────────────────┐
            │ Multi-Configuration │
            │      Project        │
            └─────────┬──────────┘
                      │
        ┌─────────────┴──────────────┐
        │       Configuration Axis    │
        │   e.g., REGISTRY = 2 values │
        └─────────────┬──────────────┘
                      │
         ┌────────────┴────────────┐
         │                           │
┌────────▼────────┐       ┌─────────▼────────┐
│  Matrix Run 1   │       │   Matrix Run 2    │
│ REGISTRY=dockerhub │     │ REGISTRY=ghpackages │
└────────┬──────────┘       └────────┬──────────┘
         │                             │
         │  (same build steps repeated)│
         ▼                             ▼
  Build, Test, Deploy           Build, Test, Deploy
```

---

# **Conclusion**

A **Multi-Configuration Project** is a powerful Jenkins feature that allows systematic, parallel execution of builds across defined variable combinations. It is excellent for **cross-environment testing**, **multi-platform validation**, and **multi-target deployments**—making it valuable for DevOps training and matrix-style automation scenarios.



# LAB - MULTI REGISTRY PUSHING USING MATRIX PROJECT

## Prerequisites (machine & accounts)

1. A running Jenkins server (recommended LTS).
2. Docker installed on the machine where Jenkins executes the build (either the Jenkins master if it runs builds, or an agent node). Verify:

   ```
   docker --version
   ```
3. Jenkins must be able to run Docker commands as the `jenkins` user:

   ```sh
   sudo usermod -aG docker jenkins
   sudo systemctl restart jenkins
   ```
4. GitHub account and Docker Hub account.
5. A GitHub repository containing at least a `Dockerfile` (see example below).

# Repository layout (example)

Create a Git repo (public or private). Minimal files:

```
/multi-registry-demo
  ├─ Dockerfile

Example `Dockerfile`:

```Dockerfile
FROM alpine:latest
CMD ["echo", "Multi registry deployment successful"]
```

Push the repo to GitHub.

---

# Create required credentials

You will create (A) Docker Hub credentials, (B) GitHub PAT (for GHCR), and (C) optional Git credentials for cloning if repository is private.

## A — Docker Hub credentials

**Type:** *Username with password*

1. Login to Jenkins as an administrator.
2. **Manage Jenkins → Manage Credentials → (select domain: Global) → Add Credentials**.
3. Choose:

   * **Kind:** `Username with password`
   * **Username:** `<your-dockerhub-username>`
   * **Password:** `<your-dockerhub-password/PAT>`
   * **ID:** `dockerhub-creds` (use this exact id or pick your own but note it)
   * **Description:** `Docker Hub username/password`
4. Click **OK**.

> Note: Docker Hub still supports username/password for `docker login`. If you use an access token, put token in the password field and username as your Docker Hub username.

## B — GitHub Packages credentials (GHCR)

**Option 1:** store as `Username with password` where the “password” is a Personal Access Token (PAT).

### Create a PAT on GitHub:

1. On GitHub: **Settings → Developer settings → Personal access tokens**.

   * If using **Tokens (classic)**: Generate new token with scopes:

     * `write:packages`, `read:packages`, and if you will push to private repos add `repo`.
   * If using **Fine-grained tokens**, ensure token has package write permission and appropriate repo access. (Classic tokens are simpler for this lab.)
2. Copy the token (you will not be able to see it again).

### Add to Jenkins:

1. **Manage Jenkins → Manage Credentials → Global → Add Credentials**
2. **Kind:** `Username with password`

   * **Username:** `<your-github-username>`
   * **Password:** `<your-github-PAT>`
   * **ID:** `github-creds`
   * **Description:** `GitHub username and PAT for GHCR`
3. Click **OK**.

> GHCR login uses `docker login ghcr.io -u USERNAME --password-stdin` where `--password-stdin` reads the PAT.


---

# Ensure Jenkins has the Credentials Binding plugin

1. **Manage Jenkins → Manage Plugins → Installed** → search for **Credentials Binding Plugin**.
2. If not installed, install from **Available** tab and restart Jenkins.

This plugin will let you inject credentials as environment variables inside the build.

---

# Create the Multi-Configuration Project

1. Jenkins **Dashboard → New Item**
2. **Enter name:** `multi-registry-deploy`
3. **Select:** *Multi-configuration project* (Matrix Project) → **OK**

## Job configuration — Source Code Management

* Under **Source Code Management → Git**:

  * **Repository URL:** `https://github.com/<your-username>/multi-registry-demo.git`
  * **Credentials:** select `git-ssh-creds` or use no credentials for public repo.
* Branch: `*/main` (or appropriate branch)

## Add Configuration Axis

* Scroll to **Configuration Matrix** section → **Add Axis → User-defined Axis**

  * **Name:** `REGISTRY`
  * **Values:** `dockerhub ghpackages`
  * This creates two runs: one with `REGISTRY=dockerhub` and the other with `REGISTRY=ghpackages`.

## Configure Build Environment → Credentials binding

We will bind the two credentials (`dockerhub-creds`, `github-creds`) to environment variables.

1. **Build Environment** → Check **Use secret text(s) or file(s)** (or **Credentials Binding** depending on Jenkins version).
2. Click **Add** → **Username and password (separated)** (Credentials Binding types vary slightly by Jenkins version; you may see *Username & password*, *Username/password (separated)* etc.)

   * **Credentials:** select `dockerhub-creds`
   * **Username Variable:** `DOCKERHUB_USER`
   * **Password Variable:** `DOCKERHUB_PASS`
3. Click **Add** → **Username and password (separated)** again:

   * **Credentials:** select `github-creds`
   * **Username Variable:** `GITHUB_USER`
   * **Password Variable:** `GITHUB_TOKEN`
4. If you need Git credentials for private repo clone and used `Username with password` for git, optionally bind them as `GIT_USER` / `GIT_PASS`. (Not necessary if you configured the Git SCM credential directly.)

> When binding username/password credentials this way, Jenkins will automatically set the environment variables for the build run — the values are masked in logs by default.

---

# Add Build Step (Execute shell) — full script with explanations

Under **Build → Add build step → Execute shell**, paste the script below. This script uses the bound env vars and the `REGISTRY` axis to decide where to push.

```bash
#!/bin/bash
set -euo pipefail

IMAGE_NAME="multi-registry-demo"
TAG="v1"
WORKDIR="$WORKSPACE"  # Jenkins workspace

echo "Workspace: $WORKDIR"
echo "Registry axis: $REGISTRY"

if [ "$REGISTRY" = "dockerhub" ]; then
  # Docker Hub: use DOCKERHUB_USER / DOCKERHUB_PASS
  echo "Logging in to Docker Hub as $DOCKERHUB_USER"
  echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin

  IMAGE_TAG="$DOCKERHUB_USER/$IMAGE_NAME:$TAG"
  echo "Building $IMAGE_TAG"
  docker build -t "$IMAGE_TAG" "$WORKDIR"

  echo "Pushing $IMAGE_TAG"
  docker push "$IMAGE_TAG"

  echo "Docker Hub push complete: $IMAGE_TAG"
fi

if [ "$REGISTRY" = "ghpackages" ]; then
  # GitHub Container Registry (ghcr.io) uses GH username + PAT
  echo "Logging in to GHCR (ghcr.io) as $GITHUB_USER"
  echo "$GITHUB_TOKEN" | docker login ghcr.io -u "$GITHUB_USER" --password-stdin

  IMAGE_TAG="ghcr.io/$GITHUB_USER/$IMAGE_NAME:$TAG"
  echo "Building $IMAGE_TAG"
  docker build -t "$IMAGE_TAG" "$WORKDIR"

  echo "Pushing $IMAGE_TAG"
  docker push "$IMAGE_TAG"

  echo "GHCR push complete: $IMAGE_TAG"
fi
```

### Notes on the script

* Uses `docker login` with credentials passed via stdin to avoid exposing tokens.
* Uses `$WORKSPACE` (Jenkins environment variable) as the build context.
* The image naming convention:

  * Docker Hub: `<dockerhub-username>/<image>:<tag>`
  * GHCR: `ghcr.io/<github-username>/<image>:<tag>`
* If you want different tags per build (e.g., commit SHA), you can generate `TAG=$(git rev-parse --short HEAD)` or use `${BUILD_NUMBER}`.

---

# Save and run

1. Click **Save**.
2. On job page: **Build Now**.
3. The matrix will launch two child builds (one per axis). You can watch each sub-build console log.

---

# Verifications

## Docker Hub

* Go to `https://hub.docker.com/r/<dockerhub-username>/` and verify `multi-registry-demo` repository has tag `v1`.
* If repo doesn’t show: verify `docker push` succeeded in Jenkins console.

## GitHub Packages (GHCR)

* On GitHub, go to `https://github.com/<github-username>?tab=packages` or to the repo’s **Packages** tab. For GHCR, image will be visible under your packages or in **[https://ghcr.io/v2/](https://ghcr.io/v2/)<user>/**.
* You can also run locally:

  ```sh
  docker pull ghcr.io/<github-username>/multi-registry-demo:v1
  docker pull <dockerhub-username>/multi-registry-demo:v1
  ```

---

# Common issues & troubleshooting

1. **Permission denied during `docker` command**

   * Ensure `jenkins` user is in `docker` group and Jenkins was restarted.
   * Or run the job on an agent that has Docker and appropriate permissions.

2. **`docker login` fails for GHCR**

   * Ensure the PAT scopes include `write:packages` and `read:packages`. If image is in a private repo, include `repo` scope.
   * Check token validity and that username is correct.

3. **Build fails due to missing Dockerfile**

   * Confirm repository root contains `Dockerfile` or change `docker build` path.

4. **Credentials not available / empty env vars**

   * Ensure Credentials Binding plugin is installed and you correctly selected the credentials in the binding.
   * Check that the binding maps to the variable names used in the script (`DOCKERHUB_USER`, etc.).

5. **Images not visible on Docker Hub or GHCR**

   * Check `docker push` output — it will show a success line.
   * For GHCR, images may be private by default; check package visibility settings in GitHub and whether token user can view it.

6. **Rate limits or 401 errors pushing to Docker Hub**

   * Docker Hub may enforce rate limits for anonymous pulls; pushing requires correct login. If you see 401 — re-check credentials.

---

# Optional improvements and security best practices

1. **Use SSH for Git** (recommended for private repos) and configure `SSH Username with private key` credential in Jenkins. Assign it to the SCM in the job, not via bindings.
2. **Use short-lived tokens or rotate PATs** regularly.
3. **Use Jenkins agent container†**: run builds inside a Docker-in-Docker or Docker socket-backed agent to isolate builds.
4. **Use a declarative Jenkinsfile** stored in your repo for reproducible CI. (I can provide a Pipeline example if you want.)
5. **Tag images with build metadata**: use `TAG=${GIT_COMMIT::7}` or `TAG=${BUILD_NUMBER}` for traceability.
6. **Avoid printing secrets** to console; bindings mask them but avoid `echo $GITHUB_TOKEN`.

---

# Example: mapping summary (IDs & env vars)

| Jenkins Credential ID |                                     Kind | Bound Vars                         |
| --------------------- | ---------------------------------------: | ---------------------------------- |
| `dockerhub-creds`     |                   Username with password | `DOCKERHUB_USER`, `DOCKERHUB_PASS` |
| `github-creds`        | Username with password (PAT as password) | `GITHUB_USER`, `GITHUB_TOKEN`      |

Use these exact variable names in the build script.

