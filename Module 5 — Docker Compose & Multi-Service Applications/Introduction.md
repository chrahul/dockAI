

#  **Module 5 — Docker Compose & Multi-Service Applications**

### *From Single Containers to Complete, Connected Stacks*


<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/b0903b45-ef82-4632-b9cf-b2c865ec619e" />


---

##  **Context**

Until now, each module taught you one layer of Docker’s ecosystem:

| Module | Focus                           | Key Takeaway                                             |
| ------ | ------------------------------- | -------------------------------------------------------- |
| 1      | Foundations of Containerization | What containers are & why they changed DevOps            |
| 2      | Images & Build System           | How containers are built, cached, and secured            |
| 3      | Lifecycle & Runtime Behavior    | How containers live, restart, and interact with the host |
| 4      | Storage & Persistence           | How containers remember and store data                   |

But **real applications** are never a single container.
They’re a network of multiple services — web front-ends, databases, caches, queues — all talking to each other.

This is where **Docker Compose** enters the picture.
It lets you define, run, and scale multi-container applications using one declarative YAML file.

---

##  **Why Docker Compose Matters**

In modern development, reproducibility and consistency are everything.
Compose bridges the gap between **developer laptops** and **production environments** by:

* Describing the entire stack as code (`docker-compose.yml`)
* Managing dependencies (DB → API → Frontend) automatically
* Handling networks, volumes, and restart policies seamlessly
* Making multi-service setups **one-command deployable**

> “If Docker makes apps portable,
> Docker Compose makes systems portable.”

---

##  **Topics Covered in Module 5**

### **Lesson 20 – Docker Compose Fundamentals**

* Anatomy of `docker-compose.yml`
* Services, networks, volumes, and dependencies
* Bringing up and tearing down stacks with `up`, `down`, `ps`

### **Lesson 21 – Networking & Service Discovery**

* Default bridge networks in Compose
* Container DNS names & linking services
* Port mapping patterns for local development

### **Lesson 22 – Environment Management & Profiles**

* `.env` files and variable substitution
* Compose profiles (dev, staging, prod)
* Secrets and configuration best practices

### **Lesson 23 – Healthchecks & Dependency Control**

* Startup order vs readiness
* `depends_on` and health conditions
* Auto-restarts for self-healing services

### **Lesson 24 – Volumes & Persistence in Compose**

* Shared volumes between services
* Data containers vs named volumes
* Database backups and migration strategies

### **Lesson 25 – Scaling & Multi-Container Patterns**

* `docker compose up --scale`
* Load balancing and round-robin connections
* Stateless service design for horizontal scale

### **Lesson 26 – Compose in CI/CD Workflows**

* Using Compose in Jenkins & GitHub Actions
* Test environments and ephemeral stacks
* Deploying Compose to remote servers (SSH, Swarm)

### **Lesson 27 – Complete Project: Multi-Service Stack**

* Deploy a full web application (Nginx + Flask + Redis + MySQL)
* Persist data across restarts
* Add monitoring and logging containers

---

##  **How It Connects to Previous Modules**

* You’ll **reuse** Dockerfiles (Module 2) to build images for each service.
* You’ll **apply** runtime concepts (Module 3) to manage health and resource limits.
* You’ll **attach** volumes (Module 4) for persistent storage.
* You’ll **combine them all** into a declarative stack using Compose.

By the end of this module, you’ll be able to spin up a complete multi-container application — **database, backend, frontend, cache, and monitoring — all in one command.**


