
#  **Module 8 – Docker Security, SBOM & CI/CD Integration**

## **Lesson 35 – Container Security Fundamentals**

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/fe4eb7b9-c3bf-4b00-a6b9-8eed112d41bd" />


---

##  **Learning Objective**

Understand the **core security principles behind Docker and containers** — how they isolate processes, manage privileges, and secure the runtime environment using **namespaces, cgroups, capabilities, and the Docker daemon model**.
This is where we build a true DevSecOps mindset — securing from the **inside out**, not just patching from the outside.

---

##  **1. Why Container Security Matters**

In modern DevOps, containers are everywhere — but security is often an afterthought.
A single vulnerable image or misconfigured container can expose an entire cluster.
Container security is about understanding how Docker isolates, enforces boundaries, and controls what a container *can and cannot* do.

> Security starts at the build, not at production.

You can’t bolt on security later — it must be baked into your **image**, **runtime**, and **pipeline**.

---

##  **2. How Docker Enforces Isolation**

Docker containers aren’t virtual machines — they’re **processes** running on the same kernel.
So, isolation isn’t achieved through hardware virtualization, but through **kernel-level mechanisms**.

### Key Building Blocks of Container Security:

| **Mechanism**                       | **Purpose**                      | **Example / Tool**                      |
| ----------------------------------- | -------------------------------- | --------------------------------------- |
| **Namespaces**                      | Isolate what a process can *see* | PID, NET, IPC, UTS, MNT, USER           |
| **Control Groups (cgroups)**        | Limit what a process can *use*   | CPU, memory, I/O limits                 |
| **Capabilities**                    | Restrict what a process can *do* | Drop `NET_ADMIN`, `SYS_ADMIN`, etc.     |
| **Seccomp (Secure Computing Mode)** | Filter allowed syscalls          | Custom profiles via `--security-opt`    |
| **AppArmor / SELinux**              | Enforce access control policies  | Mandatory Access Control for containers |
| **Rootless Mode**                   | Run Docker daemon as non-root    | `dockerd-rootless.sh`                   |

---

##  **3. Namespaces: The Foundation of Container Isolation**

Namespaces create **separate views of system resources** for each container:

* **PID Namespace:** Isolates process IDs (each container has its own process tree)
* **NET Namespace:** Provides isolated network stacks, interfaces, and routes
* **MNT Namespace:** Controls mount points and filesystems
* **UTS Namespace:** Isolates hostname and domain name
* **USER Namespace:** Maps user IDs inside container to unprivileged users on host

Example:

```bash
docker run -it --rm alpine sh
ps aux
```

You’ll notice that processes inside the container start from PID 1 — isolated from the host.

---

##  **4. cgroups: Controlling Resource Usage**

cgroups ensure **one container doesn’t hog system resources**.

Example:

```bash
docker run -d --cpus="1.0" --memory="512m" nginx
```

This limits the container to 1 CPU core and 512 MB RAM.
Perfect for multi-tenant environments or preventing resource exhaustion attacks.

---

##  **5. Capabilities and Privilege Management**

By default, Docker containers **don’t run as full root** — Docker drops many Linux capabilities automatically.
You can inspect them using:

```bash
docker run --rm alpine capsh --print
```

Drop additional privileges:

```bash
docker run --rm --cap-drop=ALL nginx
```

Only add back what you need:

```bash
docker run --cap-add=NET_BIND_SERVICE nginx
```

**Principle of Least Privilege (POLP)** — give each container *only* the permissions it needs.

---

##  **6. Runtime Security Layers**

| **Layer**         | **Security Control**                     | **Goal**                         |
| ----------------- | ---------------------------------------- | -------------------------------- |
| **Host OS**       | Hardened kernel, minimal attack surface  | Protect from container escape    |
| **Docker Daemon** | TLS auth, restricted socket access       | Prevent remote abuse             |
| **Image Build**   | Verified base images, multi-stage builds | Prevent injection during build   |
| **Runtime**       | Capabilities, AppArmor, seccomp          | Prevent privilege escalation     |
| **Network**       | Firewalls, namespaces, and overlays      | Isolate traffic between services |

---

##  **7. Practical Security Tips**

 Use **official base images** from trusted registries
 Run containers as **non-root users**
 Regularly scan images for vulnerabilities (`trivy`, `grype`)
 Avoid using `--privileged` unless absolutely necessary
 Keep Docker and OS patched
 Limit network exposure using `--publish` carefully
 Use **read-only file systems** for immutable containers
 Configure **resource limits** (CPU, memory, I/O)

---

##  **8. Hands-On Lab: Secure Your First Container**

### Step 1 — Run an unprivileged container

```bash
docker run -it --user 1000:1000 --read-only alpine sh
```

### Step 2 — Scan your image

```bash
trivy image nginx:latest
```

### Step 3 — Drop all capabilities

```bash
docker run --rm --cap-drop=ALL nginx
```

### Step 4 — Enable seccomp

```bash
docker run --security-opt seccomp=default.json nginx
```

---

##  **9. Real-World Analogy**

Think of Docker security like an apartment building:

* **Namespaces** → separate rooms
* **cgroups** → rent limit (can’t use more than allocated)
* **Capabilities** → restricted privileges (no key to other rooms)
* **Seccomp / AppArmor** → building rules (what actions are allowed)

Together, they make containers safe roommates sharing the same house (kernel).

---

##  **10. Summary**

| **Concept**            | **Key Takeaway**                                  |
| ---------------------- | ------------------------------------------------- |
| **Isolation**          | Achieved through namespaces and cgroups           |
| **Control**            | Capabilities, seccomp, AppArmor restrict behavior |
| **Least Privilege**    | Drop unnecessary permissions                      |
| **Proactive Security** | Scan, patch, and restrict from the start          |

---

##  **Next Lesson Preview – Lesson 36**

> **Vulnerability Scanning & Image Hardening**
> Learn how to find, fix, and prevent vulnerabilities in container images using tools like **Trivy** and **Grype**, and apply image-hardening best practices.

