

#  **Lesson 27 – Registry Authentication & Access Control**

---

##  **Learning Objective**

Learn how Docker registries handle **authentication**, **authorization**, and **secure communication**, ensuring that only trusted users can push or pull container images — whether on **Docker Hub**, **private registries**, or **enterprise systems** like **ECR**, **ACR**, or **Harbor**.

---

##  **1. Context: Why Security Matters in Registries**

Registries are at the heart of every CI/CD workflow — they store your **runtime artifacts**.
If an attacker gains access, they can:

* Push a **malicious image** under your organization name.
* **Overwrite tags** like `latest` with compromised versions.
* Steal **proprietary images** or secrets embedded within.

Hence, registry security = **image supply-chain security**.

---

##  **2. Concept: How Authentication Works in Docker Registries**

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/7863921c-271f-4458-988b-37729a6ac4f5" />


Docker clients authenticate before pushing or pulling private images.

###  Authentication Flow

1. `docker login <registry>` → client sends credentials.
2. Registry verifies via `htpasswd`, token, or SSO.
3. Access token stored in `~/.docker/config.json`.
4. Subsequent `docker push/pull` requests reuse the token.

###  Common Auth Methods

| Registry Type                   | Supported Methods                                  |
| ------------------------------- | -------------------------------------------------- |
| Docker Hub                      | Username + Password / PAT (Personal Access Token)  |
| Private Registry (`registry:2`) | Basic Auth via htpasswd / TLS Cert                 |
| AWS ECR                         | Temporary token via AWS CLI (`get-login-password`) |
| Azure ACR                       | Azure AD Service Principal / Managed Identity      |
| Harbor                          | Local Users + LDAP/AD + OIDC + RBAC                |

---

##  **3. Hands-On Lab: Secure a Private Registry with Basic Auth**

###  **Goal**

Enable authentication on your self-hosted `registry:2` instance.

---

### **Step 1 – Create Password File**

```bash
mkdir auth
docker run --entrypoint htpasswd httpd:2 -Bbn admin MySecurePass > auth/htpasswd
```

This creates a bcrypt-hashed password file.

---

### **Step 2 – Run Registry with Auth Enabled**

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

 Registry now requires credentials.

---

### **Step 3 – Login**

```bash
docker login localhost:5000
Username: admin
Password: MySecurePass
```

Docker stores this in `~/.docker/config.json`:

```json
{
  "auths": {
    "localhost:5000": {
      "auth": "YWRtaW46TXlTZWN1cmVQYXNz"
    }
  }
}
```

---

### **Step 4 – Push and Pull Images**

```bash
docker tag nginx:latest localhost:5000/secure-nginx:v1
docker push localhost:5000/secure-nginx:v1
docker pull localhost:5000/secure-nginx:v1
```

Works only for authenticated users.

---

### **Step 5 – Enable TLS (HTTPS)**

```bash
mkdir certs && openssl req -newkey rsa:4096 -nodes -sha256 \
  -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt
```

Run registry with SSL:

```bash
docker run -d \
  -p 443:5000 \
  --name registry-ssl \
  -v "$(pwd)"/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry:2
```

Then trust the certificate locally:

```bash
sudo mkdir -p /etc/docker/certs.d/localhost:5000
sudo cp certs/domain.crt /etc/docker/certs.d/localhost:5000/ca.crt
```

---

##  **4. Access Control & RBAC (Enterprise Registries)**

### Harbor Example

* **Project-based RBAC:** Admins, Developers, Guests.
* **LDAP/AD Integration:** Centralized identity.
* **Robot Accounts:** For CI/CD automation with scoped permissions.
* **Policy Controls:** Image signing, vulnerability scans, and content trust.

### ECR / ACR / OCIR

* Leverage **cloud IAM** roles and policies.
* Define granular permissions: `PushImage`, `PullImage`.
* Integrate with CICD using temporary tokens.

---

##  **5. Registry Security Checklist**

| Control                                | Purpose                     |
| -------------------------------------- | --------------------------- |
| HTTPS only                           | Encrypt traffic to registry |
| Auth tokens                          | Avoid plain passwords       |
| RBAC & IAM                           | Limit who can push/pull     |
| Immutable tags                       | Prevent overwrites          |
| Signed images (Notary/Content Trust) | Verify integrity            |
| Vulnerability scans                  | Detect CVEs in base images  |

---

##  **6. Visual Workflow**

```
┌──────────────┐
│ docker login │──► Registry verifies user ──► Token issued
└──────┬───────┘
       ▼
┌──────────────────────┐
│ docker push/pull     │──► Auth validated each time
└──────────────────────┘
```

---

##  **7. Summary**

| Concept        | Key Takeaway                                     |
| -------------- | ------------------------------------------------ |
| Authentication | Verifies user identity                           |
| Authorization  | Controls what actions are allowed                |
| HTTPS & Certs  | Protect data in transit                          |
| RBAC/IAM       | Granular role management                         |
| Best Practice  | Never allow anonymous push to private registries |

---

##  **Next Lesson Preview – Lesson 28**

> **“Image Promotion Pipelines (CI/CD Integration)”**
> See how registries connect with CI/CD tools (Jenkins, GitHub Actions, GitLab CI) to automate build-push-promote workflows across environments.

