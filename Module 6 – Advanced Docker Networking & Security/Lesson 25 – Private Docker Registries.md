

# **Module 6 – Docker Registries & Image Promotion**

## **Lesson 25 – Private Docker Registries (Docker Hub & Self-Hosted)**

---

##  **Learning Objective**

Understand how Docker images are stored and distributed through registries, how to set up your **own private registry**, and how authentication, tagging, and pushing/pulling work securely in a real-world environment.

---

##  **1. Context: Why We Need Registries**

In any real-world container workflow, developers build images locally — but deployment happens on remote servers, clusters, or clouds.
A **Docker Registry** acts as the **central warehouse** for all your images.

Think of it like **GitHub for container images**:

| Role       | Git Equivalent  | Docker Equivalent            |
| ---------- | --------------- | ---------------------------- |
| Repository | Git repo        | Image repository             |
| Commit     | Code commit     | Image version / tag          |
| Push/Pull  | `git push/pull` | `docker push/pull`           |
| Registry   | GitHub/GitLab   | Docker Hub, ECR, ACR, Harbor |

Every production-grade CI/CD pipeline depends on registries to:

* Store versioned builds
* Share images between developers and environments
* Securely authenticate image pulls during deployment
* Promote tested images across environments (Dev → QA → Prod)

---

##  **2. Concept: What Is a Docker Registry**


<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/396bd1a5-f6fe-4751-80b2-a68cff2f7a46" />


###  **Registry vs Repository**

* **Registry** → the server or service that hosts multiple repositories (e.g., Docker Hub, Amazon ECR).
* **Repository** → a collection of tagged images of the same application.

Example:

```
registry: docker.io
repository: rahulch/nginx
tags: latest, v1.0, v1.1
```

###  **Public Registries**

* **Docker Hub** (default for most users)
* **GitHub Container Registry (GHCR)**
* **Google Artifact Registry**, **Azure ACR**, **AWS ECR**

### 🔹 **Private Registries**

* Useful for organizations that need control, security, or air-gapped environments.
* Options:

  * Self-hosted open-source registry (`registry:2` image)
  * Enterprise solutions (Harbor, JFrog Artifactory, Nexus)
  * Managed registries (ECR/ACR/OCIR)

---

##  **3. How Docker Uses Registries**

When you run:

```bash
docker pull nginx:latest
```

Docker resolves it as:

```
docker.io/library/nginx:latest
```

When you push a custom image:

```bash
docker push myrepo/app:1.0
```

Docker checks:

* Are you logged in (`docker login`)?
* Do you have permission to push?
* Does the registry exist or is it private?

If yes, Docker layers are uploaded sequentially, and metadata is registered in the registry’s manifest.

---

##  **4. Hands-On Lab: Deploy a Private Docker Registry**

###  **Lab Overview**

You’ll deploy your own **private registry** using Docker itself, push an image to it, and pull it back — demonstrating full local lifecycle.

---

### **Step 1 — Run a Private Registry Container**

```bash
docker run -d \
  -p 5000:5000 \
  --name registry \
  registry:2
```

 This runs the official open-source registry image listening on port **5000**.

---

### **Step 2 — Verify It’s Running**

```bash
docker ps
```

You’ll see:

```
CONTAINER ID   IMAGE       PORTS                    NAMES
abc123         registry:2  0.0.0.0:5000->5000/tcp   registry
```

---

### **Step 3 — Tag a Local Image for Your Registry**

Assume you already have `nginx:latest` locally:

```bash
docker tag nginx:latest localhost:5000/nginx:v1
```

---

### **Step 4 — Push Image to the Private Registry**

```bash
docker push localhost:5000/nginx:v1
```

Output:

```
The push refers to repository [localhost:5000/nginx]
v1: digest: sha256:e01d… size: 1363
```

Your registry now stores this image.

---

### **Step 5 — Verify by Pulling It Back**

First remove the local copy:

```bash
docker rmi localhost:5000/nginx:v1
```

Then pull it again:

```bash
docker pull localhost:5000/nginx:v1
```

If successful →  Your private registry works!

---

### **Step 6 — Browse Registry Contents**

You can check repositories stored inside:

```bash
curl http://localhost:5000/v2/_catalog
```

Output:

```json
{"repositories":["nginx"]}
```

---

##  **5. Securing the Private Registry**

Running it plain HTTP is fine for local labs, but production requires **HTTPS + Authentication**.

**Option 1: Use Self-Signed Certificate**

```bash
mkdir -p certs && openssl req \
  -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key \
  -x509 -days 365 -out certs/domain.crt
```

Run registry with SSL:

```bash
docker run -d \
  -p 443:5000 \
  --name secure-registry \
  -v "$(pwd)"/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry:2
```

**Option 2: Add Basic Authentication**

```bash
mkdir auth && docker run --entrypoint htpasswd \
  httpd:2 -Bbn admin MySecurePass > auth/htpasswd
```

Run:

```bash
docker run -d \
  -p 5000:5000 \
  --name registry-auth \
  -v "$(pwd)"/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
  registry:2
```

Then login before pushing:

```bash
docker login localhost:5000
```

---

##  **6. Visual Overview**

```
             ┌────────────────────────────┐
             │        Developer PC        │
             │                            │
             │  docker build → tag → push │
             └─────────────┬──────────────┘
                           │
                           ▼
             ┌────────────────────────────┐
             │  Private Registry (Port 5000) │
             │  Stores versioned images       │
             │  with optional Auth & SSL      │
             └─────────────┬──────────────┘
                           │
                           ▼
             ┌────────────────────────────┐
             │   Other Systems / CI/CD    │
             │ docker pull app:v1         │
             └────────────────────────────┘
```

---

##  **7. Troubleshooting Tips**

| Problem                                         | Cause                  | Fix                                                  |
| ----------------------------------------------- | ---------------------- | ---------------------------------------------------- |
| `connection refused`                            | Wrong port mapping     | Check port `5000`                                    |
| `x509: certificate signed by unknown authority` | Using self-signed cert | Add cert to Docker trust store `/etc/docker/certs.d` |
| `unauthorized: authentication required`         | Missing login          | Run `docker login <registry>`                        |

---

##  **8. Summary**

| Concept                | Key Takeaway                                             |
| ---------------------- | -------------------------------------------------------- |
| **Docker Registry**    | Central service to store, version, and distribute images |
| **Public Registry**    | Docker Hub / GHCR – global sharing                       |
| **Private Registry**   | Self-hosted or enterprise-managed for internal use       |
| **Registry Image**     | `registry:2` official image for self-hosting             |
| **Push/Pull Workflow** | Tag → Push → Pull – similar to Git                       |
| **Security**           | Always enforce HTTPS & authentication in production      |

---

##  **Next Lesson Preview – Lesson 26**

> **“Image Tagging & Versioning Strategies”**
> Learn how to tag images across dev/QA/prod pipelines, avoid the `latest` trap, and establish a proper promotion workflow.


