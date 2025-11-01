

#  **Lesson 20 — Docker Compose Fundamentals**

### *From single containers to fully defined application stacks*

---

##  **Context**

In previous modules, we’ve learned how to build and run containers using `docker build` and `docker run`.
That’s perfect when you’re managing **one container at a time**.

But modern applications are made of **multiple interconnected containers**:

* A backend API container
* A frontend web container
* A database container
* A cache (like Redis)
* A monitoring sidecar

Starting and linking all of them manually with `docker run` is painful and error-prone.
Enter **Docker Compose** — the orchestration tool that lets you define your entire multi-container setup in one file.

> “If Docker is the engine, Docker Compose is the blueprint.”

---

##  **Concept: What is Docker Compose?**

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/084e2709-fd01-4efc-be3b-c565a7f14833" />


Docker Compose is a **tool for defining and running multi-container Docker applications** using a single declarative YAML file (`docker-compose.yml`).

It helps you:

* Manage multiple services in one place
* Define networks and shared volumes
* Set environment variables, restart policies, and dependencies
* Reproduce your stack on any machine with a single command

---

###  **Core Components of docker-compose.yml**

Here’s the basic structure:

```yaml
version: "3.9"

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"

  db:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: appdb
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

###  Breakdown:

* **services:** Defines each container and its configuration
* **networks:** (optional) Connects containers together
* **volumes:** Creates persistent storage for containers
* **depends_on:** Controls startup order
* **environment:** Injects configuration variables

---

##  **Hands-On Lab 1 — Running Your First Compose Stack**

### Step 1: Create a directory

```bash
mkdir compose-demo && cd compose-demo
```

### Step 2: Create a `docker-compose.yml`

```bash
nano docker-compose.yml
```

Paste this:

```yaml
version: "3.9"
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

### Step 3: Run the stack

```bash
docker compose up -d
```

### Step 4: Check status

```bash
docker compose ps
```

### Step 5: Stop and remove

```bash
docker compose down
```

---

##  **Hands-On Lab 2 — Multi-Service Stack**

Now, let’s add a database and a backend:

```yaml
version: "3.9"
services:
  api:
    image: python:3.10-slim
    working_dir: /app
    command: python -m http.server 5000
    ports:
      - "5000:5000"
    volumes:
      - .:/app
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: appdb
    volumes:
      - pg_data:/var/lib/postgresql/data

volumes:
  pg_data:
```

Run it:

```bash
docker compose up -d
```

Check:

```bash
docker compose ps
docker compose logs api
```

---

##  **What Happens Behind the Scenes**

When you run `docker compose up`, the following sequence occurs:

1. Docker Compose **reads the YAML** and creates an **isolated network**.
2. It starts each **service as a container**.
3. Shared volumes and environment variables are attached.
4. Each container can **discover others by service name** (e.g., `db:5432`).

You’ve now created a **self-contained micro-environment** — something you can replicate anywhere.

---

##  **Key Takeaways**

| Concept          | Description                                                     |
| ---------------- | --------------------------------------------------------------- |
| **Compose File** | YAML file defining your stack (services, networks, volumes)     |
| **Service**      | A running container defined inside the compose file             |
| **Network**      | Automatic bridge network enabling inter-container communication |
| **Volume**       | Persistent storage shared between services                      |
| **Commands**     | `docker compose up`, `down`, `ps`, `logs`, `restart`, `stop`    |

---

##  **Next Lesson → Lesson 21: Networking & Service Discovery**

We’ll explore **how containers find and talk to each other** inside Compose — including DNS-based service discovery, port mapping, and custom bridge networks.

---

