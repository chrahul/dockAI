

#  Lesson 4: Docker Architecture – Daemon, CLI, and Components

### *(Module 1 – Foundations of Docker and Containerization)*

---

##  Context

When you type this simple command:

```bash
docker run nginx
```

…a **lot** happens behind the scenes.
Docker downloads the image, creates a container, isolates it using Linux namespaces, applies cgroups for resource limits, sets up networking, and finally runs your app.

To understand Docker deeply, we must break down its **internal architecture** — how all these moving parts talk to each other.

> Think of Docker as a mini operating system for containers — with its own client, server, and runtime layers.

---

##  Concept: The Docker Engine Architecture

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/3ae72c40-ac0d-4d3d-89fc-fd4946b46d3b" />


Docker Engine follows a **client–server model** consisting of three key components:

1. **Docker CLI (Client)**

   * The command-line interface (`docker run`, `docker ps`, etc.)
   * Sends REST API calls to the Docker Daemon.
   * You can run it locally or remotely.

2. **Docker Daemon (dockerd)**

   * The brain of Docker.
   * Handles image pulls, container creation, networks, volumes.
   * Exposes a REST API over Unix socket (`/var/run/docker.sock`) or TCP.
   * Manages the container lifecycle.

3. **Container Runtime (containerd & runc)**

   * `containerd`: Manages container execution, images, and storage.
   * `runc`: Low-level runtime that actually spawns and manages containers via Linux namespaces and cgroups (OCI-compliant).

---

###  The Control Flow (Step-by-Step)

When you run:

```bash
docker run nginx
```

Here’s what happens behind the curtain:

| Step | Component      | Description                                                     |
| ---- | -------------- | --------------------------------------------------------------- |
| 1️  | **CLI**        | Sends REST API call to Docker Daemon via `/var/run/docker.sock` |
| 2️  | **Daemon**     | Checks if `nginx` image exists locally                          |
| 3️  | **Daemon**     | If not found, pulls image from Docker Hub (registry)            |
| 4️  | **Daemon**     | Uses **containerd** to create a container snapshot              |
| 5️  | **containerd** | Invokes **runc** to spawn the process with namespaces & cgroups |
| 6️  | **runc**       | Runs the isolated process (container)                           |
| 7️  | **Daemon**     | Attaches I/O, manages lifecycle (start, stop, logs, etc.)       |

> The CLI itself doesn’t create containers — it just talks to the Docker Daemon.

---

##  Key Components Summary

| Layer                          | Description                                            |
| ------------------------------ | ------------------------------------------------------ |
| **Docker CLI**                 | Sends commands to the daemon                           |
| **Docker Daemon (dockerd)**    | Core process managing images, containers, and networks |
| **containerd**                 | Manages container lifecycle                            |
| **runc**                       | Creates containers using Linux kernel features         |
| **Image Registry**             | Stores and distributes Docker images                   |
| **Storage Drivers (overlay2)** | Manage layered file systems                            |
| **Network Drivers**            | Bridge, host, none, overlay                            |

---

##  Real-World Analogy

Imagine running a **restaurant kitchen**:

| Role             | Docker Equivalent         | Description                              |
| ---------------- | ------------------------- | ---------------------------------------- |
| Waiter           | **Docker CLI**            | Takes your order (`docker run nginx`)    |
| Kitchen Manager  | **Docker Daemon**         | Coordinates cooking, staff, and workflow |
| Chef             | **containerd/runc**       | Actually cooks (executes the container)  |
| Grocery Supplier | **Registry (Docker Hub)** | Provides ingredients (images)            |

---

##  Hands-On Lab 4 – Inspecting Docker Internals

### **Goal:**

Explore how Docker CLI, Daemon, and Runtime interact.

---

### **Step 1: Check Docker Processes**

```bash
ps -ef | grep dockerd
```

Output example:

```
/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

---

### **Step 2: Check containerd**

```bash
ps -ef | grep containerd
```

---

### **Step 3: Run a Container**

```bash
docker run -d --name test-nginx nginx
```

---

### **Step 4: Locate Container Files**

```bash
sudo ls /var/lib/docker/containers
```

Each folder = a unique container ID.

---

### **Step 5: Inspect OCI Configuration**

```bash
sudo cat /var/lib/docker/containers/<container_id>/config.v2.json | jq .
```

You’ll see:

* Environment variables
* Entrypoint
* Network settings
* Resource limits

---

### **Step 6: Observe Runtime Calls**

```bash
sudo ls /run/containerd/io.containerd.runtime.v2.task/moby/
```

You’ll find container process metadata managed by containerd and runc.

---

##  Key Takeaways

* Docker Engine is built on **modular layers**: CLI → Daemon → containerd → runc.
* The **CLI** is just a client; the **Daemon** does the real work.
* **containerd** manages containers, and **runc** actually runs them.
* Everything in Docker runs through **REST APIs** and **Linux kernel features**.

---

**Next Lesson → [Lesson 5: Docker Images, Layers, and Union File System](../Lesson-5-Docker-Images-and-Layers.md)**

---


