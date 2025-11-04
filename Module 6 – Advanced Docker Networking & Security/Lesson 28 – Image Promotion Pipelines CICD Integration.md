#  **Lesson 28 – Image Promotion Pipelines (CI/CD Integration)**

---

##  **Learning Objective**

Understand how Docker images flow through automated **CI/CD pipelines**, moving seamlessly from build to test to production using controlled **promotion stages** — ensuring that only validated images are deployed in production environments.

---

##  **1. Context: Why Automate Image Promotion**

Manual image promotion (tagging → pushing → promoting) works fine for labs,
but in enterprises — it doesn’t scale.

A DevOps pipeline automates this process by:

* Building images automatically from source code
* Running tests before pushing to registry
* Promoting only **verified** images between environments
* Maintaining **traceability** and **immutability** of each version

This ensures reliability, repeatability, and auditability in your deployment process.

---

## **2. Concept: Image Promotion in CI/CD**

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/2b68f0f0-49da-4879-9a78-dd048321c13e" />


###  Basic Flow

1. **Build Stage** – Build Docker image from source.
2. **Test Stage** – Run container-based tests (unit, integration, vulnerability).
3. **Push Stage** – Push image to Dev registry.
4. **Promotion Stage** – Tag and push same image digest to QA or Prod registry.

All of this happens **automatically** when triggered via commit, PR, or tag.

---

###  Promotion Rules

| **Rule**                                    | **Description**                           |
| ------------------------------------------- | ----------------------------------------- |
| Promote only signed images                  | Ensures authenticity                      |
| Use immutable digests                       | Avoids tag overwrite issues               |
| Trigger promotion via events                | Example: tag creation or pipeline success |
| Separate registries or namespaces per stage | Isolation of environments                 |
| Integrate security scanning                 | Block vulnerable builds                   |

---

##  **3. Hands-On Lab: Build → Push → Promote Pipeline**

###  **Goal**

Automate a build and promotion workflow using **GitHub Actions** (works similarly in Jenkins or GitLab CI).

---

### **Step 1 — Create Repository Secrets**

Add the following GitHub repository secrets:

```
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
```

---

### **Step 2 — Define CI/CD Workflow**

File: `.github/workflows/docker-build-promote.yml`

```yaml
name: Docker Build & Promote

on:
  push:
    branches:
      - main
      - release/*

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and Push Image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ secrets.DOCKERHUB_USERNAME }}/myapp:${{ github.sha }}
            ${{ secrets.DOCKERHUB_USERNAME }}/myapp:dev

  promote:
    runs-on: ubuntu-latest
    needs: build
    if: startsWith(github.ref, 'refs/heads/release/')
    
    steps:
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Promote Image to Prod
        run: |
          docker pull ${{ secrets.DOCKERHUB_USERNAME }}/myapp:dev
          docker tag ${{ secrets.DOCKERHUB_USERNAME }}/myapp:dev \
                     ${{ secrets.DOCKERHUB_USERNAME }}/myapp:prod
          docker push ${{ secrets.DOCKERHUB_USERNAME }}/myapp:prod
```

---

### **Step 3 — Verify on Docker Hub**

After a successful pipeline run:

```bash
docker pull rahulch/myapp:dev
docker pull rahulch/myapp:prod
```

Both tags will point to the same **digest**, ensuring controlled promotion.

---

##  **4. Enterprise Example: Jenkins Declarative Pipeline**

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t myapp:${BUILD_NUMBER} .'
            }
        }
        stage('Push Dev') {
            steps {
                sh 'docker tag myapp:${BUILD_NUMBER} myregistry/myapp:dev'
                sh 'docker push myregistry/myapp:dev'
            }
        }
        stage('Promote to QA') {
            when { branch 'release/*' }
            steps {
                sh 'docker pull myregistry/myapp:dev'
                sh 'docker tag myregistry/myapp:dev myregistry/myapp:qa'
                sh 'docker push myregistry/myapp:qa'
            }
        }
        stage('Promote to Prod') {
            when { branch 'main' }
            steps {
                sh 'docker pull myregistry/myapp:qa'
                sh 'docker tag myregistry/myapp:qa myregistry/myapp:prod'
                sh 'docker push myregistry/myapp:prod'
            }
        }
    }
}
```

---

##  **5. CI/CD Integration Patterns**

| **Tool**            | **Integration Type**               | **Notes**                     |
| ------------------- | ---------------------------------- | ----------------------------- |
| **GitHub Actions**  | Native Docker actions              | Easy to use, built-in secrets |
| **Jenkins**         | Docker pipeline plugin             | Flexible, enterprise-ready    |
| **GitLab CI**       | Built-in registry + promotion jobs | Strong native integration     |
| **Azure DevOps**    | Container job + ACR tasks          | Perfect for ACR               |
| **ArgoCD + Harbor** | GitOps-style promotion             | Declarative and secure        |

---

##  **6. Visual Workflow**

```
┌─────────────┐
│  Build App  │
└──────┬──────┘
       ▼
┌─────────────┐
│ Test Image  │
└──────┬──────┘
       ▼
┌──────────────┐
│ Push to Dev  │
└──────┬───────┘
       ▼
┌──────────────┐
│ Promote → QA │
└──────┬───────┘
       ▼
┌──────────────┐
│ Promote → Prod │
└──────────────┘
```

---

##  **7. Summary**

| Concept                    | Key Takeaway                                 |
| -------------------------- | -------------------------------------------- |
| **Image Promotion**        | Automate moving images between environments  |
| **CI/CD Integration**      | Build → Test → Push → Promote                |
| **Immutable Tags**         | Prevent overwriting and ensure traceability  |
| **Event-driven Pipelines** | Trigger promotion on success or tag creation |
| **Security Gates**         | Block vulnerable or unsigned images          |

---

## **Next Lesson Preview – Lesson 29**

> **“Optimizing and Mirroring Registries”**
> Learn how to improve performance, caching, and redundancy with registry mirrors and garbage collection — preparing you for production-scale environments.

