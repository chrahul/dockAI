#  **Lesson 26 — Docker Compose in Production**

### *Taking Your Multi-Service Stack from Laptop to Live Environment*

---

##  **Context**

You’ve learned to **build**, **link**, and **scale** containers using Docker Compose in development.
But deploying those same stacks to *production* — where uptime, monitoring, and resilience matter — requires a new level of understanding.

This lesson bridges that gap between **developer-friendly Compose** and **production-ready orchestration**.

> “Development is about convenience.
> Production is about confidence.”

---

##  **Concept: From Compose to Production**



<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/0c70e65b-ef84-4445-8cf0-cdb3b49a1871" />


Compose is not just a local tool — it’s the **blueprint** for your system.
You can take the same `docker-compose.yml` file and:

* Run it on a **staging server**
* Deploy it to **multiple nodes using Docker Swarm**
* Import it into **Portainer, ECS, or Kubernetes (via Kompose)**

The goal: **Same definition, different environment.**

---

##  **Example: Three-Stage Deployment Setup**

### **1. Local (Developer)**

Fast feedback, bind mounts, no resource limits.

```bash
docker compose --profile dev up
```

### **2. Staging**

Test environment variables, replicas, and volumes.

```bash
docker compose --profile staging up -d
```

### **3. Production**

Strict limits, monitoring, persistent storage.

```bash
docker compose --profile prod up -d
```

Each stage uses the same Compose file with different **.env** values and **profiles**.

---

##  **Production-Grade docker-compose.yml Example**

```yaml
version: "3.9"

services:
  web:
    image: nginx:alpine
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.5"
          memory: 256M
    ports:
      - "80:80"
    networks:
      - frontend

  app:
    image: myorg/app:latest
    environment:
      - ENV=prod
    depends_on:
      db:
        condition: service_healthy
    networks:
      - frontend
      - backend

  db:
    image: mysql:8
    environment:
      - MYSQL_ROOT_PASSWORD=rootpass
      - MYSQL_DATABASE=appdb
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h localhost -p$$MYSQL_ROOT_PASSWORD"]
      interval: 10s
      retries: 5
      timeout: 5s

volumes:
  db_data:

networks:
  frontend:
  backend:
```

 **Key production-grade elements included:**

* Healthchecks
* Restart policies
* CPU and memory limits
* Separate networks for isolation
* Persistent volumes for data

---

##  **Lab 1 — Running Compose in Production**

### Step 1: Copy files to the server

```bash
scp -r myapp/ ubuntu@prod-server:/opt/myapp/
```

### Step 2: Connect and start

```bash
ssh ubuntu@prod-server
cd /opt/myapp
docker compose up -d
```

### Step 3: Validate

```bash
docker compose ps
docker compose logs -f
docker inspect <container_id> | grep IPAddress
```

---

##  **Lab 2 — Auto-Restart and Healthcheck Validation**

Simulate a crash:

```bash
docker kill <container_id>
```

Watch it come back:

```bash
docker ps
```

Docker will automatically restart the unhealthy container if you used:

```yaml
restart_policy:
  condition: on-failure
```

---

##  **Lab 3 — Running with Docker Swarm**

Convert your Compose file directly for multi-host deployments:

```bash
docker swarm init
docker stack deploy -c docker-compose.yml mystack
```

Inspect:

```bash
docker stack services mystack
docker stack ps mystack
```

 Swarm uses Compose syntax but runs containers **across multiple nodes** with built-in load balancing.

---

##  **Pro Tip: Portainer Integration**

You can visually manage your Compose stacks with **Portainer** — a GUI for Docker & Swarm.

* Access at: `http://<server_ip>:9443`
* Upload your `docker-compose.yml`
* Monitor containers, volumes, and networks from dashboard.

---

##  **Best Practices for Production Compose**

| Category       | Recommendation                                         |
| -------------- | ------------------------------------------------------ |
| **Security**   | Use non-root users, secrets, and `.env` files securely |
| **Storage**    | Use named volumes or external cloud storage            |
| **Networking** | Separate internal & external networks                  |
| **Monitoring** | Enable logging drivers (json-file, fluentd, Loki)      |
| **Resilience** | Set restart policies & healthchecks                    |
| **Scaling**    | Use replicas & external load balancers (e.g., Traefik) |
| **CI/CD**      | Use GitHub Actions or Jenkins to deploy automatically  |

---

##  **Key Takeaways**

* `docker compose` isn’t just for dev — it can run staging and prod stacks
* Combine **profiles**, **healthchecks**, and **resource limits** for resilience
* Use **Swarm, ECS, or Portainer** for distributed production
* Focus on **monitoring, scaling, and secure configs**

---

##  **Next Lesson → Lesson 27: CI/CD with Docker Compose**

We’ll integrate Compose with **GitHub Actions** and **Jenkins**, automating image builds, testing, and deployments — from commit to container.

---

