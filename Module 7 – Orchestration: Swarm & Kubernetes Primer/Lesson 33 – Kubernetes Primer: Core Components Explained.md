#  **Module 7 – Orchestration: Swarm & Kubernetes Primer**

## **Lesson 33 – Kubernetes Primer: Core Components Explained**

---

##  **Learning Objective**

Understand the **architecture and core components of Kubernetes (K8s)** — including Pods, Nodes, Control Plane, Scheduler, and Controller — and how it provides **declarative, self-healing orchestration** at scale across clusters.

---

##  **1. Context: From Swarm to Kubernetes**

While **Docker Swarm** is simple and tightly integrated with Docker,
**Kubernetes** evolved as a **planet-scale orchestration system** originally built by Google (inspired by its internal Borg system).

Kubernetes focuses on:

* Declarative infrastructure (“desired state”)
* Cluster-wide automation
* Self-healing, scaling, and scheduling
* Extensibility via APIs and controllers

Today, it’s the **de facto standard** for container orchestration across all major clouds — AWS (EKS), Azure (AKS), GCP (GKE), and Oracle OCI (OKE).

---

##  **2. Concept: What Is Kubernetes?**

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/06c443a7-b76c-48cb-9bf2-cd5db1535a9e" />


At its core, Kubernetes is:

> “A system that automates the deployment, scaling, and management of containerized applications.”

It abstracts away individual machines and treats your entire infrastructure as one big pool of compute resources.

You define **what you want to run**, and Kubernetes ensures **it runs exactly that way** — always.

---

##  **3. Kubernetes Architecture Overview**

###  **High-Level View**

```
┌──────────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                        │
│                                                              │
│  ┌────────────────────────┐       ┌────────────────────────┐ │
│  │     Control Plane      │       │       Worker Nodes     │ │
│  │  (Brain of K8s)        │       │  (Run Application Pods)│ │
│  └────────────────────────┘       └────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

Kubernetes is divided into **two main planes**:

| Plane             | Description                                   |
| ----------------- | --------------------------------------------- |
| **Control Plane** | Manages the cluster — decides what runs where |
| **Worker Plane**  | Executes the actual workloads (containers)    |

---

##  **4. Key Components**

###  **A. Control Plane Components**

| Component                                          | Description                                                                                 |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| **API Server (`kube-apiserver`)**                  | The front door of Kubernetes — all requests go through here (CLI, UI, or other components). |
| **Scheduler (`kube-scheduler`)**                   | Assigns Pods to Nodes based on resources, affinity, and policies.                           |
| **Controller Manager (`kube-controller-manager`)** | Ensures cluster state matches the desired state (e.g., starts new pods if one fails).       |
| **etcd**                                           | The distributed key-value database storing cluster configuration and state.                 |
| **Cloud Controller Manager**                       | Integrates with underlying cloud APIs (e.g., AWS, Azure, OCI).                              |

---

###  **B. Node (Worker Plane) Components**

| Component             | Description                                                              |
| --------------------- | ------------------------------------------------------------------------ |
| **kubelet**           | Agent running on each node — ensures containers are running as expected. |
| **kube-proxy**        | Handles network routing and service load balancing.                      |
| **Container Runtime** | Responsible for running containers (Docker, containerd, CRI-O).          |

---

###  **C. Core Logical Objects**

| Object                 | Purpose                                                          |
| ---------------------- | ---------------------------------------------------------------- |
| **Pod**                | Smallest deployable unit — wraps one or more containers.         |
| **Deployment**         | Defines how many Pods should run, manages updates and rollbacks. |
| **Service**            | Provides stable networking and load-balancing for Pods.          |
| **Namespace**          | Logical partition for organizing resources (Dev, QA, Prod).      |
| **ConfigMap / Secret** | Externalize configuration and sensitive data.                    |

---

##  **5. Hands-On Lab: Create a Local Kubernetes Cluster**

###  Tools

You can use **Minikube** or **Kind (Kubernetes in Docker)**.

#### **Option 1: Using Minikube**

```bash
minikube start --driver=docker
```

Check cluster status:

```bash
kubectl get nodes
```

Output:

```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   2m    v1.29.0
```

#### **Option 2: Using Kind**

```bash
kind create cluster --name dockai
```

Verify:

```bash
kubectl get pods -A
```

---

### **Step 1 — Create a Deployment**

```bash
kubectl create deployment web --image=nginx
```

### **Step 2 — Scale the Deployment**

```bash
kubectl scale deployment web --replicas=3
```

### **Step 3 — Expose as a Service**

```bash
kubectl expose deployment web --type=NodePort --port=80
```

### **Step 4 — Verify**

```bash
kubectl get svc
```

Output:

```
NAME         TYPE       CLUSTER-IP     PORT(S)        AGE
web          NodePort   10.96.200.10   80:31000/TCP   1m
```

 Kubernetes automatically created Pods, assigned them to nodes, and exposed the service — with self-healing and load-balancing built-in.

---

##  **6. Control Loop Concept**

Kubernetes continuously runs a **control loop**:

```
Desired State (YAML)  →  Actual State (Cluster)
```

If a pod crashes, the controller automatically replaces it.

This is the **heart of declarative orchestration**.

---

##  **7. Visual Architecture Summary**

```
┌──────────────────────────────┐
│        API Server            │
│         │                    │
│  ┌──────┴───────────┐        │
│  │Scheduler         │        │
│  │Controller Manager│        │
│  │etcd (State DB)   │        │
└──┴──────┬───────────┘────────┘
          │
          ▼
   ┌────────────────────┐
   │     Worker Node    │
   │ kubelet + proxy    │
   │ Runs Pod(s)        │
   └────────────────────┘
```

---

##  **8. Summary**

| Concept               | Key Takeaway                                       |
| --------------------- | -------------------------------------------------- |
| **Kubernetes**        | Industry-standard container orchestration platform |
| **Control Plane**     | Manages cluster state and scheduling               |
| **Worker Plane**      | Executes workloads (Pods)                          |
| **Pods**              | Smallest unit that runs one or more containers     |
| **Declarative Model** | “You tell what, K8s decides how”                   |
| **Self-Healing**      | Automatically replaces failed containers           |

---

##  **Next Lesson Preview – Lesson 34**

> **Kubernetes vs Swarm – Key Differences & Use Cases**
> We’ll compare both orchestrators side by side — architecture, scalability, networking, and real-world adoption.

