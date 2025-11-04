
#  **Module 7 – Orchestration: Swarm & Kubernetes Primer**

## **Lesson 32 – Deploying & Scaling Services in Swarm**

---

##  **Learning Objective**

Learn how to **deploy, scale, update, and roll back** containerized services in Docker Swarm clusters.
You’ll see how Swarm automatically manages container lifecycles, maintains desired state, and balances traffic across replicas — with zero manual intervention.

---

##  **1. Context: Why Services, Not Containers**

In a single-host Docker setup, we use:

```bash
docker run -d nginx
```

This runs one container — simple, but not scalable or self-healing.

In **Swarm**, we instead create **services**:

```bash
docker service create --name web --replicas 3 nginx
```

Now, Swarm ensures:

* 3 containers (tasks) are running at all times
* Distributed across available nodes
* Load-balanced internally
* Recovered automatically if one fails

 The **service**, not the container, becomes the atomic unit of management.

---

##  **2. Concept: What Is a Docker Service**

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/d736cae0-c4f9-44ec-b904-1dc2ea17680b" />


A **service** in Swarm represents a desired state:

> “Run *N replicas* of this image, expose *this port*, attach *to this network*.”

### 🔹 Core Elements

| Component         | Description                                           |
| ----------------- | ----------------------------------------------------- |
| **Service**       | High-level definition (e.g., web app with 5 replicas) |
| **Task**          | A running container instance (created by the service) |
| **Replicas**      | Number of tasks defined in the service                |
| **Desired State** | The target configuration Swarm must maintain          |

Swarm continuously **reconciles** the desired and actual state — like a “self-healing” container fabric.

---

##  **3. Hands-On Lab: Deploy and Scale Services**

###  **Goal**

Deploy a replicated Nginx service, scale it up, perform a rolling update, and verify load balancing.

---

### **Step 1 — Create a Service**

```bash
docker service create --name web --replicas 3 -p 8080:80 nginx
```

Check the running services:

```bash
docker service ls
```

```
ID      NAME  MODE        REPLICAS  IMAGE
abcd123 web   replicated  3/3       nginx:latest
```

---

### **Step 2 — Inspect Service and Tasks**

```bash
docker service ps web
```

Example output:

```
ID       NAME     IMAGE     NODE        DESIRED STATE
t1a3b4   web.1    nginx     worker-1    Running
t1a3b5   web.2    nginx     worker-2    Running
t1a3b6   web.3    nginx     manager     Running
```

 Each replica runs on a different node.

---

### **Step 3 — Access the Service**

Access from any Swarm node:

```
http://<node-ip>:8080
```

Swarm’s internal load balancer automatically distributes requests among replicas.

---

### **Step 4 — Scale the Service**

```bash
docker service scale web=6
```

Output:

```
web scaled to 6
```

Now check:

```bash
docker service ls
```

```
ID      NAME  MODE        REPLICAS  IMAGE
abcd123 web   replicated  6/6       nginx:latest
```

Swarm will automatically deploy additional containers across nodes based on resource availability.

---

### **Step 5 — Perform a Rolling Update**

```bash
docker service update --image nginx:1.25 web
```

Output:

```
web: update in progress
web: update completed
```

Swarm performs **rolling updates**, replacing one replica at a time to prevent downtime.

---

### **Step 6 — Rollback (if needed)**

If the update fails or causes issues:

```bash
docker service rollback web
```

Swarm will revert to the previous stable configuration automatically.

---

##  **4. Service Update Strategies**

| Option                          | Description                            | Example                  |
| ------------------------------- | -------------------------------------- | ------------------------ |
| `--update-delay`                | Time gap between updates               | `--update-delay 10s`     |
| `--update-parallelism`          | Number of tasks updated simultaneously | `--update-parallelism 2` |
| `--rollback`                    | Rollback on failure                    | `--rollback`             |
| `--limit-cpu`, `--limit-memory` | Resource limits                        | `--limit-memory 256M`    |

Example:

```bash
docker service update \
  --image nginx:1.25 \
  --update-delay 5s \
  --update-parallelism 2 \
  web
```

---

##  **5. Visual Workflow**

```
                ┌────────────────────────┐
                │     Manager Node       │
                │ (Desired State: 3 Repl.)│
                └───────────┬────────────┘
                            │
                ┌───────────┴───────────┐
                │       Scheduler       │
                └───────────┬───────────┘
                            │
        ┌──────────────┬──────────────┬──────────────┐
        ▼              ▼              ▼
   ┌──────────┐   ┌──────────┐   ┌──────────┐
   │ Worker-1 │   │ Worker-2 │   │ Worker-3 │
   │ web.1    │   │ web.2    │   │ web.3    │
   └──────────┘   └──────────┘   └──────────┘
```

Swarm ensures:

* Desired replicas are always running
* Automatic load-balancing between tasks
* Rolling updates and rollbacks are safe

---

##  **6. Key Swarm Service Commands**

| Command                    | Purpose                |
| -------------------------- | ---------------------- |
| `docker service create`    | Deploy a new service   |
| `docker service ls`        | List running services  |
| `docker service ps <name>` | View running tasks     |
| `docker service scale`     | Adjust replica count   |
| `docker service update`    | Change image/config    |
| `docker service rollback`  | Revert to last version |
| `docker service rm`        | Remove service         |

---

##  **7. Summary**

| Concept             | Key Takeaway                                      |
| ------------------- | ------------------------------------------------- |
| **Service**         | Logical abstraction for containerized workloads   |
| **Task**            | Running container managed by Swarm                |
| **Scaling**         | Swarm dynamically adds/removes replicas           |
| **Rolling Updates** | Safe, automated deployment mechanism              |
| **Rollback**        | Built-in recovery from failures                   |
| **Load Balancing**  | Automatic distribution of traffic across replicas |

---

##  **Next Lesson Preview – Lesson 33**

> **Kubernetes Primer – Core Components Explained**
> Understand Kubernetes architecture — Pods, Nodes, Scheduler, Controller, and how it differs from Swarm in managing large-scale containerized systems.

