
#  **Module 7 – Orchestration: Swarm & Kubernetes Primer**

## **Lesson 31 – Docker Swarm Mode: Architecture & Setup**

---

##  **Learning Objective**

Learn what **Docker Swarm Mode** is, how it differs from standalone Docker, and how to **build, manage, and deploy** a Swarm cluster with managers, workers, and overlay networks — the foundation for orchestrating services at scale.

---

##  **1. Context: From Docker Engine to Docker Swarm**

Docker Swarm transforms ordinary Docker engines into a **cluster of cooperating nodes**.
Instead of running containers individually, you manage **services** that the Swarm automatically schedules and scales across available machines.

Think of it as:

> “Docker Engine + Distributed Brain + Self-Healing.”

---

##  **2. What Is Docker Swarm Mode?**

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/97ad4e38-a222-4239-acad-c9da59967bea" />


Swarm mode is **native clustering** built into Docker (no extra installation).
It allows multiple Docker hosts to act as a single, unified cluster.

###  Key Roles

| Role             | Description                                                  |
| ---------------- | ------------------------------------------------------------ |
| **Manager Node** | Controls the Swarm, schedules tasks, maintains cluster state |
| **Worker Node**  | Executes container tasks assigned by the manager             |
| **Service**      | A group of containers performing the same function           |
| **Task**         | A single running container instance (replica)                |

---

##  **3. Swarm Architecture Overview**

```
                ┌──────────────────────────────┐
                │        Manager Node          │
                │   - Cluster state            │
                │   - Scheduling decisions     │
                │   - Replication control      │
                └──────────────┬───────────────┘
                               │
                     Overlay Network
                               │
   ┌───────────────────────────┼───────────────────────────┐
   │                           │                           │
┌───────┐                 ┌───────┐                 ┌───────┐
│Worker1│                 │Worker2│                 │Worker3│
│Runs   │                 │Runs   │                 │Runs   │
│Tasks  │                 │Tasks  │                 │Tasks  │
└───────┘                 └───────┘                 └───────┘
```

###  Communication:

* **Manager ↔ Worker** → via `Docker API` over **port 2377**
* **Overlay Networking** → allows container-to-container communication across nodes
* **Raft Consensus** → ensures consistent cluster state among managers

---

##  **4. Hands-On Lab: Initialize and Explore Swarm**

###  **Goal**

Create a **Swarm cluster**, join worker nodes, and deploy your first replicated service.

---

### **Step 1 — Initialize Swarm Mode**

On the first node (manager):

```bash
docker swarm init --advertise-addr <manager-IP>
```

Output:

```
Swarm initialized: current node (abc123) is now a manager.
To add a worker to this swarm, run the following command:

docker swarm join --token SWMTKN-1-xyz <manager-IP>:2377
```

---

### **Step 2 — Add Worker Nodes**

On other nodes:

```bash
docker swarm join --token SWMTKN-1-xyz <manager-IP>:2377
```

Check nodes:

```bash
docker node ls
```

Output:

```
ID     HOSTNAME     STATUS   AVAILABILITY   MANAGER STATUS
abc123 manager-node Ready    Active         Leader
def456 worker-1     Ready    Active
ghi789 worker-2     Ready    Active
```

---

### **Step 3 — Deploy a Service**

Let’s deploy Nginx with 3 replicas:

```bash
docker service create --name web --replicas 3 -p 8080:80 nginx
```

Check status:

```bash
docker service ls
```

```
ID      NAME  MODE      REPLICAS  IMAGE
1abc2d  web   replicated 3/3      nginx:latest
```

---

### **Step 4 — Inspect Tasks**

```bash
docker service ps web
```

You’ll see containers distributed across multiple nodes:

```
ID       NAME     IMAGE     NODE        DESIRED STATE
xyz123   web.1    nginx     worker-1    Running
abc234   web.2    nginx     worker-2    Running
pqr345   web.3    nginx     manager     Running
```

 Congratulations — your first Swarm cluster is live and load-balanced automatically.

---

##  **5. Overlay Networking in Swarm**

Docker Swarm automatically creates an **overlay network** that spans all nodes.

```bash
docker network create -d overlay mynet
```

Deploy service using this network:

```bash
docker service create --name app --network mynet nginx
```

This enables cross-node communication between containers — without manual IP configuration.

---

##  **6. Swarm Service Lifecycle**

| Action       | Command                                        | Description                |
| ------------ | ---------------------------------------------- | -------------------------- |
| **Scale Up** | `docker service scale web=5`                   | Add replicas               |
| **Update**   | `docker service update --image nginx:1.25 web` | Rolling update             |
| **Rollback** | `docker service rollback web`                  | Revert to previous version |
| **Remove**   | `docker service rm web`                        | Clean up service           |

---

##  **7. Key Features of Swarm Mode**

| Feature               | Description                                     |
| --------------------- | ----------------------------------------------- |
| **Native to Docker**  | No extra tools or setup                         |
| **Declarative model** | Define desired state (replicas, networks, etc.) |
| **Rolling updates**   | Built-in zero downtime updates                  |
| **Service discovery** | Internal DNS for inter-container communication  |
| **Security**          | TLS-based node communication and certificates   |
| **High Availability** | Manager quorum ensures fault tolerance          |

---

##  **8. Summary**

| Concept             | Key Takeaway                                     |
| ------------------- | ------------------------------------------------ |
| **Docker Swarm**    | Native orchestration built into Docker           |
| **Manager Node**    | Schedules and monitors tasks                     |
| **Worker Node**     | Executes container workloads                     |
| **Service**         | Logical group of replicated containers           |
| **Overlay Network** | Connects services across nodes                   |
| **Use Case**        | Lightweight orchestration for Docker-only setups |

---

##  **Next Lesson Preview – Lesson 32**

> **Deploying & Scaling Services in Swarm**
> Learn how to manage service lifecycles — scaling, updating, rolling back, and load-balancing containers dynamically across your cluster.

