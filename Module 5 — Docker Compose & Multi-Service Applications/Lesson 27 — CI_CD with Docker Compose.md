# **Lesson 27 — CI/CD with Docker Compose**

### *Automating Build, Test, and Deploy Workflows — From Commit to Container*

---

##  **Context**

So far, you’ve learned how to define and scale multi-service applications using Docker Compose.
But in a real DevOps setup, running `docker compose up` manually isn’t enough.

We need automation — something that builds, tests, and deploys containers **the moment code changes**.

That’s where **CI/CD pipelines** come in.

> “Automation is not about replacing effort.
> It’s about eliminating inconsistency.”

---

##  **Concept: CI/CD with Docker Compose**

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/b2360567-56e7-402d-b009-404d8615bc58" />


Docker Compose simplifies running complex multi-container environments locally.
By integrating it with **GitHub Actions** or **Jenkins**, we can:

* Automatically build Docker images on every commit.
* Run tests in isolated containers.
* Push versioned images to a registry.
* Deploy updated containers to servers or staging environments.

---

## **Workflow Overview**

```
Git Commit → CI/CD Pipeline → Build → Test → Push → Deploy
```

1. Developer commits code to GitHub.
2. GitHub Actions (or Jenkins) triggers a pipeline.
3. Docker builds the new images.
4. Containers run for integration testing.
5. Successful builds are pushed to a registry (Docker Hub, ECR, ACR).
6. Production server pulls the new image and updates the stack via Compose.

---

##  **Example: GitHub Actions Workflow**

File: `.github/workflows/docker-compose-ci.yml`

```yaml
name: CI/CD with Docker Compose

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and Push Images
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: rahulch/app:latest

      - name: Deploy via SSH using Compose
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ubuntu
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /opt/myapp
            docker compose pull
            docker compose up -d
```

This workflow:

* Builds your image
* Pushes it to Docker Hub
* Connects to the server via SSH
* Updates the running stack

---

##  **Lab — Local CI/CD Simulation with Jenkins**

### Step 1: Install Jenkins

```bash
docker run -d -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

### Step 2: Install Plugins

* Docker
* Docker Pipeline
* GitHub Integration

### Step 3: Create Jenkinsfile

```groovy
pipeline {
  agent any
  environment {
    REGISTRY = "docker.io/rahulch"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      steps {
        sh 'docker compose build'
      }
    }

    stage('Test') {
      steps {
        sh 'docker compose run --rm app pytest'
      }
    }

    stage('Push') {
      steps {
        sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
        sh 'docker compose push'
      }
    }

    stage('Deploy') {
      steps {
        sh 'ssh ubuntu@prod-server "cd /opt/myapp && docker compose pull && docker compose up -d"'
      }
    }
  }
}
```

### Step 4: Trigger Automatically

Add a GitHub Webhook → Jenkins triggers on every push to `main`.

---

##  **Best Practices for Compose CI/CD**

| Area              | Recommendation                                                     |
| ----------------- | ------------------------------------------------------------------ |
| **Security**      | Store credentials in GitHub Secrets or Jenkins Credentials Manager |
| **Tagging**       | Use `:commit-sha` or `:build-number` for versioning                |
| **Testing**       | Always include a `docker compose run --rm test` stage              |
| **Rollback**      | Keep the previous image version tagged as `:stable`                |
| **Monitoring**    | Use Slack or email notifications for build status                  |
| **DRY Principle** | Reuse same Compose file across Dev → Staging → Prod with profiles  |

---

##  **Bonus: Multi-Stage Build Integration**

Compose integrates perfectly with **multi-stage Dockerfiles** (from Module 2),
allowing you to:

* Build lightweight final images
* Reuse build cache layers
* Push only optimized artifacts

```bash
docker buildx build --target prod --push -t rahulch/app:latest .
```

---

##  **Key Takeaways**

* CI/CD brings automation, consistency, and speed to Compose-based projects.
* Combine GitHub Actions or Jenkins with `docker compose build` and `up`.
* Use profiles and `.env` files to separate staging and prod configs.
* Keep credentials and registry tokens secure.
* Automate rollback and notification systems.

---

##  **Next Step**

🎓 With this, **Module 5 — Docker Compose & Multi-Service Applications** is now complete.

Up next → **Module 6: Docker Registries, Image Promotion & Supply Chain Security**
We’ll dive deep into registry automation, immutable tagging, and securing the entire image lifecycle from build to production.

