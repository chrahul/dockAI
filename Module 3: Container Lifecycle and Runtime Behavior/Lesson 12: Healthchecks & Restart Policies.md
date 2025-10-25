
#  Lesson 12: Healthchecks & Restart Policies

### *(Module 3 – Container Lifecycle & Runtime Behavior)*

---

##  Context

In real-world deployments, applications crash.
Processes hang. Databases start late. APIs go down temporarily.

When that happens, your container should *heal itself* — not wait for you to log in and restart it.

Docker provides two mechanisms for this resilience:
1️ **HEALTHCHECK** → Monitors container health status.
2️ **Restart Policies** → Define how containers restart when they fail.

Together, they form the foundation of **self-healing container infrastructure**, the same principle Kubernetes uses at scale.

---

##  Learning Goals

 Understand the purpose of the Docker **HEALTHCHECK** instruction.
 Learn how to detect and handle unhealthy containers.
 Configure and test different **restart policies** (`always`, `on-failure`, etc.).
 Combine both for automatic recovery.

---

##  Concept

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/ba760f26-d50e-40fb-a5d7-499b10f9034f" />


### 1️ HEALTHCHECK Overview

By default, Docker only knows if a container’s *main process* is running.
But a process can be alive while the app inside it is **unresponsive**.

A **HEALTHCHECK** allows Docker to periodically test if your application is *actually healthy*.

**Dockerfile Example:**

```Dockerfile
FROM nginx:alpine
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD curl -f http://localhost:80/ || exit 1
```

This tells Docker:

* Run `curl -f http://localhost:80/` every 30 seconds.
* If it fails 3 times → mark container as **unhealthy**.

---

### 2️ Viewing Health Status

```bash
docker ps
```

**Output:**

```
CONTAINER ID   IMAGE          STATUS
a9b3f4          nginx:alpine   Up 2 minutes (healthy)
b8a7d9          nginx:alpine   Up 1 minute (unhealthy)
```

To inspect:

```bash
docker inspect --format='{{.State.Health.Status}}' webapp
```

---

### 3️ Restart Policies

Restart policies control what happens when a container exits or crashes.

| Policy                       | Description                          |
| ---------------------------- | ------------------------------------ |
| **no**                       | Never restart (default).             |
| **always**                   | Always restart on exit.              |
| **unless-stopped**           | Restart unless manually stopped.     |
| **on-failure[:max-retries]** | Restart only on non-zero exit codes. |

Set at run time:

```bash
docker run -d --restart unless-stopped nginx
```

---

### 4️ Combining Healthchecks + Restart Policies

For critical services:

```bash
docker run -d \
  --restart always \
  --health-cmd "curl -f http://localhost:80/ || exit 1" \
  nginx
```

Now, if your app goes unhealthy → you can monitor or trigger restarts automatically using Docker or an orchestrator.

---

### 5️ Docker’s Self-Healing Logic

Internally, Docker tracks health in `/var/lib/docker/containers/<id>/healthcheck.log`
and exposes it to:

* `docker ps` output
* `docker inspect` state
* Docker event stream (`docker events --filter event=health_status`)

When integrated with monitoring tools, this can trigger alerts or automated recovery.

---

##  Labs — Step by Step

### Lab 12.1 — Add Healthcheck to Nginx

**Dockerfile**

```Dockerfile
FROM nginx:alpine
HEALTHCHECK --interval=10s CMD curl -f http://localhost/ || exit 1
```

**Build & Run**

```bash
docker build -t web-health .
docker run -d --name web1 -p 8080:80 web-health
```

**Check**

```bash
docker ps
docker inspect web1 --format='{{.State.Health.Status}}'
```

**Output**

```
healthy
```

---

### Lab 12.2 — Simulate a Failure

Stop nginx process inside the container:

```bash
docker exec -it web1 sh -c "pkill nginx"
```

Wait ~30 seconds and check again:

```bash
docker ps
```

**Output**

```
STATUS: Up 2 minutes (unhealthy)
```

---

### Lab 12.3 — Use Restart Policy

```bash
docker run -d --restart on-failure:3 alpine sh -c "exit 1"
docker ps -a
```

**Output**

```
Restart Count: 3
Exited (1) 10 seconds ago
```

---

### Lab 12.4 — Combine Both

```bash
docker run -d \
  --restart always \
  --health-cmd "curl -f http://localhost:80/ || exit 1" \
  nginx
```

Simulate crash:

```bash
docker exec -it <id> pkill nginx
```

Docker restarts container automatically.

---

##  Notes

* Healthchecks don’t *restart* containers — they only report status.
* Combine with **restart policies** or external monitoring to take action.
* Keep checks lightweight — don’t overload the container.
* Default interval: 30s, timeout: 30s, retries: 3.
* In Kubernetes, `livenessProbe` and `readinessProbe` evolved from Docker healthchecks.

---

##  Summary

* `HEALTHCHECK` tests if your app *works*, not just *runs*.
* `Restart Policies` define how containers recover automatically.
* Together, they form Docker’s built-in **self-healing mechanism**.

---

###  Next Lesson → **Lesson 13: Resource Limits & Isolation**

We’ll explore how Docker enforces CPU, memory, and I/O limits using **cgroups v2**, and how to observe throttling in real time with `docker stats`.

