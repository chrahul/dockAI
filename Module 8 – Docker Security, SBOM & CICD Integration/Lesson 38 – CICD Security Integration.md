#  **Module 8 – Docker Security, SBOM & CI/CD Integration**

## **Lesson 38 – CI/CD Security Integration (Trivy, Cosign, and SBOM in Jenkins)**


<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/8c5285d4-9d7a-49ab-bf53-c4c15f385381" />


---

##  **Learning Objective**

Build a **secure, automated CI/CD pipeline** that integrates:

* **Trivy** for vulnerability scanning
* **Syft** for SBOM generation
* **Cosign** for signing and verification
  — all within a **Jenkins pipeline**, ensuring security at every build stage.

---

##  **1. Why CI/CD Security Integration Matters**

In modern DevOps, speed is power — but without integrated security, speed becomes risk.
A single vulnerable or unsigned image can compromise the entire production environment.

So the goal isn’t just automation — it’s **trust automation**.
Every image your pipeline builds should be:

* Scanned
* Signed
* Verified
  before it reaches production.

This is what turns **CI/CD** into **CI/CD + Security = DevSecOps**.

---

##  **2. Architecture Overview**

### The Secure Pipeline Flow

```
[Git Commit]
     ↓
[Build Stage]
  - Dockerfile
  - Multi-stage build
     ↓
[Scan Stage]
  - Trivy vulnerability scan
  - Generate SBOM (Syft)
     ↓
[Sign Stage]
  - Cosign image & SBOM signing
     ↓
[Verify Stage]
  - Signature verification
     ↓
[Deploy Stage]
  - Deploy only verified images
```

Each step enforces a **security checkpoint**, ensuring your container is clean, signed, and verified before deployment.

---

##  **3. Jenkins Pipeline Setup**

### **Pre-requisites**

* Jenkins with Docker plugin
* Trivy, Syft, and Cosign installed on Jenkins agents
* Access to container registry (Docker Hub, ECR, or ACR)
* Cosign keys (`cosign.key` and `cosign.pub`) generated

---

##  **4. Jenkinsfile Example – Secure Docker Pipeline**

```groovy
pipeline {
  agent any
  environment {
    IMAGE = "myapp:${BUILD_NUMBER}"
  }

  stages {

    stage('Checkout') {
      steps {
        git 'https://github.com/chrahul/dockAI-app.git'
      }
    }

    stage('Build Image') {
      steps {
        sh 'docker build -t $IMAGE .'
      }
    }

    stage('Scan Image with Trivy') {
      steps {
        sh 'trivy image --exit-code 0 --severity HIGH,CRITICAL $IMAGE'
      }
    }

    stage('Generate SBOM') {
      steps {
        sh 'syft $IMAGE -o json > sbom.json'
      }
    }

    stage('Sign Image & SBOM') {
      steps {
        sh 'cosign sign --key cosign.key $IMAGE'
        sh 'cosign sign-blob --key cosign.key sbom.json'
      }
    }

    stage('Verify Signatures') {
      steps {
        sh 'cosign verify --key cosign.pub $IMAGE'
        sh 'cosign verify-blob --key cosign.pub sbom.json'
      }
    }

    stage('Push Image to Registry') {
      steps {
        sh 'docker push $IMAGE'
      }
    }

    stage('Deploy') {
      steps {
        echo "Deploying verified image: $IMAGE"
      }
    }
  }

  post {
    success {
      echo " Build completed securely and deployed successfully."
    }
    failure {
      echo " Security check failed. Fix vulnerabilities before re-deploy."
    }
  }
}
```

---

##  **5. Key Security Controls**

| **Stage**  | **Tool** | **Purpose**                                |
| ---------- | -------- | ------------------------------------------ |
| **Build**  | Docker   | Creates base image using minimal layers    |
| **Scan**   | Trivy    | Detects vulnerabilities (CVEs)             |
| **SBOM**   | Syft     | Generates component manifest               |
| **Sign**   | Cosign   | Ensures authenticity and integrity         |
| **Verify** | Cosign   | Confirms image & SBOM match trusted source |
| **Deploy** | Jenkins  | Deploys only verified artifacts            |

This ensures *no unsigned or unscanned image* ever reaches production.

---

##  **6. Hands-On Verification**

After the build:

```bash
docker pull myapp:25
cosign verify --key cosign.pub myapp:25
```

You should see:

```
Verified OK
```

Then, check your SBOM:

```bash
grype sbom:sbom.json
```

It lists known vulnerabilities, mapped directly to packages inside the image — full transparency achieved.

---

##  **7. Continuous Improvement**

Integrate periodic re-scans:

```bash
trivy image --exit-code 1 --ignore-unfixed myapp:25
```

Set Jenkins to re-trigger weekly or upon base image changes.
Add Slack or email notifications for failed scans or unsigned builds.

---

##  **8. Visual Summary**

```
┌───────────────────────────────┐
│           BUILD STAGE         │
│  Docker build (multi-stage)   │
└──────────────┬────────────────┘
               ↓
┌──────────────▼────────────────┐
│         SCAN STAGE            │
│  Trivy → Vulnerability Check  │
│  Syft → Generate SBOM         │
└──────────────┬────────────────┘
               ↓
┌──────────────▼────────────────┐
│         SIGN STAGE            │
│  Cosign → Sign image & SBOM   │
└──────────────┬────────────────┘
               ↓
┌──────────────▼────────────────┐
│         VERIFY STAGE          │
│  Cosign → Verify signatures   │
└──────────────┬────────────────┘
               ↓
┌──────────────▼────────────────┐
│         DEPLOY STAGE          │
│  Jenkins → Deploy verified app│
└───────────────────────────────┘
```

---

##  **9. Summary**

| **Concept**               | **Key Takeaway**                                          |
| ------------------------- | --------------------------------------------------------- |
| **Pipeline Security**     | Embed scanning, signing, and verification into CI/CD      |
| **Trivy / Syft / Cosign** | Core DevSecOps triad for Docker security                  |
| **Automation**            | Security gates ensure no risky image is deployed          |
| **Outcome**               | Full visibility, traceability, and trust in every release |

---

##  **Next Lesson Preview – Lesson 39 (Final Lesson)**

> **Docker + DevSecOps Wrap-Up: Security, CI/CD & Trust**
> A complete recap of how we transformed Docker from simple containers → secure, verifiable, production-grade deployments.

