

#  Lesson 9: Image Security, Scanning, and SBOM

### *(Module 2 – Docker Images, Layers, and Build Fundamentals)*

---

##  Context

We’ve now mastered how Docker **builds** and **optimizes** images — but none of it matters if the images themselves are **vulnerable** or **tampered**.

In production environments, **image integrity** and **transparency** are non-negotiable.
Before deploying an image, you must know:

* What software (and which versions) it contains
* Whether those components have vulnerabilities
* Whether the image was signed and verified

This lesson bridges the world of **image building** and **supply-chain security**, using modern DevSecOps tools like:

* **Trivy** (vulnerability scanner)
* **Syft / Grype** (SBOM + scanning)
* **Cosign** (image signing and verification)
* **Docker Scout** (Docker-native vulnerability insights)

---

##  Learning Goals

 Understand what an **SBOM (Software Bill of Materials)** is and why it matters.
 Learn how to **scan images** for known vulnerabilities.
 Generate **SBOMs** with open-source tools (Trivy, Syft).
 Digitally **sign** Docker images and **verify provenance**.
 Integrate scanning and signing into CI/CD pipelines.

---

##  Concept


<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/f739b77f-022b-40e9-9109-72d8caa7cb3b" />



### 1️ Why Image Security Matters

The 2020s saw several high-profile supply-chain attacks (e.g., SolarWinds, Log4j).
Attackers compromise build systems, inject malicious dependencies, or alter container images post-build.

The DevSecOps approach is simple:

> “Know what’s inside your image, verify who built it, and ensure it hasn’t changed.”

That’s what **SBOMs, scanning, and signing** enable.

---

### 2️ What is an SBOM?

An **SBOM** (Software Bill of Materials) is a detailed inventory of all components in an image — OS packages, libraries, dependencies, and versions.

It answers:

* What’s installed?
* Which license does it use?
* Where did it come from?

SBOM formats:

| Format        | Description                              | Tools       |
| ------------- | ---------------------------------------- | ----------- |
| **SPDX**      | Open standard backed by Linux Foundation | Syft, Trivy |
| **CycloneDX** | Common in DevSecOps pipelines            | Syft, Grype |
| **JSON**      | Default output for CI pipelines          | All tools   |

---

### 3️ Vulnerability Scanning

Scanners (Trivy, Grype, Docker Scout) compare SBOM data with public vulnerability databases (like NVD).
They report CVEs (Common Vulnerabilities and Exposures) along with severity and fix versions.

---

### 4️ Image Signing & Verification

Once an image passes scanning, it should be **signed** using a private key.
Consumers (Kubernetes clusters, registries, CI/CD pipelines) can then **verify signatures** to ensure authenticity.

Tools:

* **Cosign** (part of Sigstore) – uses keyless signatures linked to identity (OIDC, GitHub Actions).
* **Notation v2** – Docker’s OCI-native signing standard.

---

##  Labs — Step by Step

> **Pre-req:** Docker installed, Trivy and Cosign binaries available (`brew install trivy cosign` or manual setup).

---

### Lab 9.1 — Scan an Image with Trivy

```bash
docker pull alpine:3.20
trivy image alpine:3.20
```

**Sample Output**

```
alpine:3.20 (alpine 3.20.1)
===========================
Total: 4 (UNKNOWN: 0, LOW: 2, MEDIUM: 1, HIGH: 1)

┌──────────┬─────────────────────┬────────┬──────────┐
│ PACKAGE  │ VULNERABILITY ID    │ SEVERITY │ FIXED IN │
├──────────┼─────────────────────┼────────┼──────────┤
│ musl     │ CVE-2024-33602      │ HIGH   │ 1.2.4-r2 │
│ busybox  │ CVE-2024-32115      │ MEDIUM │ 1.36.1-r3 │
└──────────┴─────────────────────┴────────┴──────────┘
```

---

### Lab 9.2 — Generate an SBOM (Software Bill of Materials)

Using **Syft**:

```bash
syft alpine:3.20 -o cyclonedx-json > sbom.json
```

Preview contents:

```bash
jq '.components[0:5]' sbom.json
```

**Output (trimmed):**

```json
[
  {
    "name": "musl",
    "version": "1.2.4-r1",
    "purl": "pkg:apk/alpine/musl@1.2.4-r1"
  },
  {
    "name": "busybox",
    "version": "1.36.1-r2"
  }
]
```

---

### Lab 9.3 — Sign an Image with Cosign

Generate a key pair:

```bash
cosign generate-key-pair
```

Sign image:

```bash
cosign sign --key cosign.key alpine:3.20
```

**Output**

```
Pushing signature to: index.docker.io/library/alpine:sha256-...
Successfully signed image.
```

Verify signature:

```bash
cosign verify --key cosign.pub alpine:3.20
```

**Output**

```
Verified OK
```

 The image’s digest matches its trusted signature.

---

### Lab 9.4 — Scan and Sign Combined (CI/CD Example)

In a CI pipeline (GitHub Actions, Jenkins, etc.):

```bash
trivy image --exit-code 1 --severity HIGH,CRITICAL alpine:3.20
cosign sign --key $COSIGN_KEY alpine:3.20
```

This ensures:

* Build fails if vulnerabilities are high/critical.
* Only scanned and signed images are published.

---

### Lab 9.5 — Docker Scout (Alternative Built-in Method)

Docker Scout integrates scanning directly into Docker CLI.

```bash
docker scout quickview alpine:3.20
```

**Output (summary):**

```
Image: alpine:3.20
Critical: 0 | High: 1 | Medium: 2 | Low: 5
Base image update available: alpine:3.21
```

---

##   Notes

* Use **SBOMs** for both compliance and auditing — especially in regulated environments (finance, healthcare).
* Always **pin versions** and rebuild frequently — base image updates fix known CVEs.
* Integrate **Trivy + Cosign** directly into your CI/CD before pushing to production registries.
* Never store credentials or secrets inside images.
* Use **Docker Scout dashboards** for organization-wide visibility.

---

##  Summary

* **SBOM = full transparency** of what’s inside your images.
* **Vulnerability scanning** ensures known issues are detected early.
* **Cosign** and **Notation v2** provide cryptographic proof of authenticity.
* Combined, they build a **trust chain** from build → scan → sign → deploy.

---

### **End of Module 2: Docker Images, Layers, and Build Fundamentals**

You’ve now mastered:
 Image layering (overlay2, CoW)
 Dockerfile design and caching
 Multi-stage builds and optimization
 BuildKit and Buildx for modern pipelines
 Image scanning, SBOM, and signing

Next up — **Module 3: Container Lifecycle and Runtime Behavior**, where we go inside the container at runtime to study PID 1, healthchecks, restarts, resource limits, and logging.

---

