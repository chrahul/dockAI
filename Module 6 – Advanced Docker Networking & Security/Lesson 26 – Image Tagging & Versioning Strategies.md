

#  **Lesson 26 – Image Tagging & Versioning Strategies**

---

##  **Learning Objective**

Learn how to version and tag Docker images systematically, avoid the “latest” trap, and implement environment-specific tagging (Dev → QA → Prod) to enable safe and traceable image promotion across your pipelines.

---

##  **1. Context: Why Tagging Matters**

In source code, we rely on **Git branches/tags** to identify code versions.
In containerized environments, **image tags** serve the same purpose — they define *what code and configuration is running in each environment.*

Improper tagging (like overusing `latest`) leads to:

* Unpredictable deployments (“it worked on my machine”)
* Rollback issues — you don’t know which version was deployed
* Broken CI/CD promotions (wrong image being tested or released)

Therefore, controlled **tagging and versioning** is essential for reproducibility, traceability, and compliance.

---

##  **2. Concept: Understanding Docker Tags**

Each Docker image is uniquely identified as:

```
<registry>/<repository>:<tag>
```

Examples:

```
nginx:latest
docker.io/rahulch/app:v1.0
localhost:5000/payments:qa-2025-11-04
```

If no tag is specified, Docker defaults to `:latest`.

---

###  **Best Practices**

| **Practice**             | **Example**         | **Why**                     |
| ------------------------ | ------------------- | --------------------------- |
| Use Semantic Versioning  | `v1.0.1`, `v2.3.0`  | Aligns with app releases    |
| Include Build Metadata   | `v1.0.1-build45`    | Useful for traceability     |
| Include Environment Tags | `dev`, `qa`, `prod` | Clear separation of stages  |
| Use Git Commit SHA       | `webapp:abc1234`    | Guarantees immutability     |
| Avoid `latest`           |  `latest`          | Hard to audit and reproduce |

---

###  **Tagging Strategies in the Real World**

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/9544f073-05cc-49c8-ba20-d378d03e0a10" />



#### 1️ **Single Repository, Multi-Tag**

You maintain a single repo (e.g., `myapp`) with multiple tags:

```
myapp:v1.0
myapp:qa
myapp:prod
```

#### 2️ **Multi-Repository Promotion**

Separate repositories for each stage:

```
registry.company.com/dev/myapp:v1.0
registry.company.com/qa/myapp:v1.0
registry.company.com/prod/myapp:v1.0
```

CI/CD pipelines promote the same image digest from Dev → QA → Prod without rebuilding.

---

##  **3. Hands-On Lab: Tag, Push, and Promote**

###  **Scenario**

You have built an image `app:v1` and want to:

* Tag it for **Dev**
* Push to private registry
* Promote same image to **QA**
* Finally mark it **Prod**

---

### **Step 1 — Build Local Image**

```bash
docker build -t myapp:v1 .
```

---

### **Step 2 — Tag for Dev Environment**

```bash
docker tag myapp:v1 localhost:5000/myapp:dev-v1
docker push localhost:5000/myapp:dev-v1
```

---

### **Step 3 — Promote to QA (without rebuilding)**

Pull the exact digest from registry and retag:

```bash
docker pull localhost:5000/myapp:dev-v1
docker tag localhost:5000/myapp:dev-v1 localhost:5000/myapp:qa-v1
docker push localhost:5000/myapp:qa-v1
```

 The same immutable image is now promoted to QA.

---

### **Step 4 — Promote to Production**

```bash
docker tag localhost:5000/myapp:qa-v1 localhost:5000/myapp:prod-v1
docker push localhost:5000/myapp:prod-v1
```

---

### **Step 5 — Verify All Tags**

```bash
curl http://localhost:5000/v2/myapp/tags/list
```

Output:

```json
{"name":"myapp","tags":["dev-v1","qa-v1","prod-v1"]}
```

---

##  **4. Tagging and CI/CD Pipelines**

Automate tagging inside build pipelines:

```yaml
# GitHub Actions example
- name: Tag and Push
  run: |
    docker build -t myapp:${{ github.sha }} .
    docker tag myapp:${{ github.sha }} myregistry/myapp:qa
    docker push myregistry/myapp:${{ github.sha }}
    docker push myregistry/myapp:qa
```

You can use environment variables like:

* `BUILD_ID`
* `GIT_COMMIT`
* `ENVIRONMENT`
* `DATE`
  to generate dynamic tags.

---

##  **5. Visual Workflow**

```
┌───────────────┐     ┌──────────────────────┐
│  Build Image  │──▶──│ Tag: dev-v1          │
└───────────────┘     └──────────────┬───────┘
                                     ▼
                            ┌──────────────────────┐
                            │ Push to Dev Registry │
                            └──────────────┬───────┘
                                           ▼
                                  ┌──────────────────────┐
                                  │ Promote → QA Tag     │
                                  └──────────────┬───────┘
                                                 ▼
                                        ┌──────────────────────┐
                                        │ Promote → Prod Tag   │
                                        └──────────────────────┘
```

---

##  **6. Quick Reference**

| **Tag Type**    | **Example**        | **When to Use**  |
| --------------- | ------------------ | ---------------- |
| Build Tag       | `web:v1.0-build23` | Every build      |
| Environment Tag | `web:qa`           | Per stage        |
| Commit Tag      | `web:abc1234`      | For traceability |
| Release Tag     | `web:v1.0`         | Stable releases  |

---

##  **7. Summary**

| Concept                 | Key Takeaway                               |
| ----------------------- | ------------------------------------------ |
| **Tags**                | Identify image versions (like Git tags)    |
| **Semantic Versioning** | Enables clear release structure            |
| **Promotion**           | Push same image digest across environments |
| **Automation**          | Tag dynamically in CI/CD                   |
| **Golden Rule**         |  Don’t rely on `latest`                   |

---

##  **Next Lesson Preview – Lesson 27**

> **“Registry Authentication & Access Control”**
> Learn how to secure image repositories using credentials, tokens, and RBAC to ensure only authorized users can push or pull images.

