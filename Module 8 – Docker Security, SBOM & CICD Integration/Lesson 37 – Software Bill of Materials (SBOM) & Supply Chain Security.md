#  **Module 8 – Docker Security, SBOM & CI/CD Integration**

## **Lesson 37 – Software Bill of Materials (SBOM) & Supply Chain Security**


<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/c5d83cf7-ba63-4815-9176-8f43b4e02de4" />


---

##  **Learning Objective**

Understand what an **SBOM (Software Bill of Materials)** is, why it’s central to **software supply chain security**, and how to generate, sign, and verify SBOMs for your Docker images using tools like **Syft, Trivy, and Cosign**.

---

##  **1. Why SBOM Matters**

Imagine you’re a manufacturer — you need to know every part that goes into your product.
Software is no different.

Every container image you deploy contains **hundreds of packages, libraries, and dependencies**, often pulled from public sources.
Without visibility into these components, you’re flying blind.

That’s where an **SBOM** — *Software Bill of Materials* — comes in.
It’s a **manifest** of everything inside your software or image, providing complete transparency for auditing, compliance, and security.

> “You can’t secure what you can’t see.” — SBOM solves that visibility gap.

---

##  **2. What is an SBOM?**

An SBOM is a **machine-readable inventory** of all components used in building a software artifact — including their versions, licenses, and relationships.

Example (simplified):

```json
{
  "name": "myapp",
  "version": "1.0.0",
  "components": [
    {"name": "nginx", "version": "1.25.0"},
    {"name": "openssl", "version": "3.0.8"},
    {"name": "glibc", "version": "2.31"}
  ]
}
```

Standards include:

* **SPDX** (Software Package Data Exchange)
* **CycloneDX**
* **SWID**

---

##  **3. Generating an SBOM**

### Using **Syft (by Anchore)**

Install:

```bash
curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh
```

Generate SBOM for an image:

```bash
syft nginx:latest -o json > sbom.json
```

Output example:

```
  Indexed image
  Cataloged packages      [50 packages]
  Wrote report to sbom.json
```

### Using **Trivy**

Trivy can also generate SBOMs directly:

```bash
trivy image --format cyclonedx --output sbom.json nginx:latest
```

---

##  **4. Understanding SBOM Outputs**

SBOMs include:

* **Packages** (OS + app libraries)
* **Dependencies** (relationships)
* **Licenses**
* **Checksums**
* **Source metadata**

You can visualize the SBOM as a dependency tree — mapping all packages to their source and risk.

---

##  **5. Verifying Integrity – Image Signing with Cosign**

SBOMs give transparency, but we also need **trust** — to prove the image or SBOM hasn’t been tampered with.

This is done through **image signing** using **Cosign (by Sigstore)**.

### Install Cosign

```bash
curl -sSfL https://github.com/sigstore/cosign/releases/latest/download/cosign-linux-amd64 -o cosign
chmod +x cosign && sudo mv cosign /usr/local/bin/
```

### Sign an Image

```bash
cosign sign --key cosign.key myapp:1.0
```

### Verify Signature

```bash
cosign verify --key cosign.pub myapp:1.0
```

Cosign uses **public/private key cryptography** (or keyless signing via OIDC) to ensure that the image you deploy is exactly what was built.

---

##  **6. Securing the Supply Chain – The Big Picture**

| **Stage**         | **Security Control**                  | **Goal**                      |
| ----------------- | ------------------------------------- | ----------------------------- |
| **Source Code**   | Dependency scanning                   | Prevent known CVEs            |
| **Build**         | SBOM generation                       | Full visibility of components |
| **Image Signing** | Cosign / Notary v2                    | Ensure authenticity           |
| **Registry**      | RBAC + HTTPS + Vulnerability Scanning | Protect at rest               |
| **Deployment**    | Admission Controller / Policy Engine  | Verify before run             |

Each of these steps forms a **“chain of custody”** for your software — ensuring it’s verified, traceable, and unmodified from code to container.

---

##  **7. Hands-On Lab – Generate and Verify SBOM**

### Step 1 – Generate SBOM with Syft

```bash
syft nginx:latest -o spdx-json > nginx-sbom.json
```

### Step 2 – Sign the SBOM

```bash
cosign sign --key cosign.key nginx-sbom.json
```

### Step 3 – Verify the SBOM

```bash
cosign verify --key cosign.pub nginx-sbom.json
```

### Step 4 – Scan from SBOM (using Grype)

```bash
grype sbom:nginx-sbom.json
```

 This allows you to find vulnerabilities **without pulling or rebuilding** the image.

---

##  **8. Integrating SBOM in CI/CD**

### GitHub Actions Example

```yaml
- name: Generate SBOM
  uses: anchore/syft-action@v1
  with:
    image: myapp:latest
    output: sbom.json

- name: Scan SBOM
  uses: anchore/scan-action@v3
  with:
    sbom: sbom.json
```

This ensures every build in your pipeline produces a **verifiable, signed SBOM**, bringing full transparency to your deployments.

---

##  **9. Visual Summary**

```
┌───────────────────────────────────────────────┐
│                 Build Stage                   │
│  - Generate SBOM (Syft / Trivy)               │
│  - Sign Image and SBOM (Cosign)               │
└───────────────────────────┬───────────────────┘
                            │
             Continuous Scanning (Grype / Trivy)
                            │
┌───────────────────────────▼───────────────────┐
│                CI/CD Pipeline                 │
│   - SBOM Verification                         │
│   - Image Signing Validation                   │
│   - Deploy Only Verified Images                │
└───────────────────────────────────────────────┘
```

---

##  **10. Summary**

| **Concept**               | **Key Takeaway**                           |
| ------------------------- | ------------------------------------------ |
| **SBOM**                  | Lists all components and dependencies      |
| **Syft / Trivy**          | Generate SBOMs in SPDX or CycloneDX        |
| **Cosign**                | Signs and verifies images/SBOMs            |
| **Supply Chain Security** | Ensures authenticity, integrity, and trust |
| **CI/CD Integration**     | Automate transparency for every deployment |

---

##  **Next Lesson Preview – Lesson 38**

> **CI/CD Security Integration (Trivy, Cosign, and SBOM in Jenkins)**
> Combine everything into a live DevSecOps pipeline — from image build to scan, sign, and deploy.

