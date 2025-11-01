

#  **Lesson 23 — Healthchecks & Dependency Control**

### *Making Your Multi-Service Stacks Self-Healing and Reliable*

---

## **Context**

In a real-world multi-container environment, **startup order** matters.
Your web app shouldn’t start before the database is ready.
Your backend shouldn’t begin until Redis or Postgres has initialized.

Docker Compose handles this using:

* **`depends_on`** — to define startup order
* **`healthcheck`** — to verify service readiness
* **restart policies** — to recover failed containers

Together, they make your stack **predictable, stable, and production-like**.

> “Dependency control isn’t just about order — it’s about readiness.”

---

## **Concept: Service Dependencies**


<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/6e617a56-a000-4e01-9b65-b0f35d4f87a7" />


### Without dependency control:

If your `app` connects to a database that takes 5 seconds to start, it may fail immediately.

###  With dependency control:

You can delay the `app` startup until the `db` is *healthy* and accepting connections.

---

###  Example: Basic depends_on

```yaml
version: "3.9"
services:
  web:
    image: nginx
    depends_on:
      - db
  db:
    image: postgres:15
```

This ensures `db` starts *before* `web`,
but it doesn’t guarantee that Postgres is *ready* to accept connections.
For that, we use a **healthcheck**.

---

##  **Concept: Healthcheck**

A healthcheck runs a command inside the container at intervals to verify service readiness.

Example:

```yaml
version: "3.9"
services:
  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

* **test:** command to check container health
* **interval:** time between checks
* **timeout:** maximum wait per check
* **retries:** number of failures before marking “unhealthy”

You can inspect status:

```bash
docker ps
```

Output:

```
CONTAINER ID   NAME        STATUS
a9f...         db          Up (healthy)
b7e...         web         Up
```

---

##  **Lab 1 — Healthcheck with Dependency**

Let’s make a self-healing stack of an app and a database.

```yaml
version: "3.9"

services:
  app:
    image: alpine
    command: sh -c "until nc -z db 5432; do echo waiting for db; sleep 2; done && echo DB Ready && sleep infinity"
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5
```

### Run:

```bash
docker compose up
```

You’ll see:

```
db_1   | database system is ready to accept connections
app_1  | DB Ready
```

 The `app` waited until `db` was actually healthy.

---

##  **Lab 2 — Restart and Self-Healing**

You can automatically restart failed services using `restart` policies.

```yaml
services:
  api:
    image: myapp:latest
    restart: unless-stopped
```

| Policy           | Description                            |
| ---------------- | -------------------------------------- |
| `no`             | Never restart                          |
| `always`         | Always restart, even after manual stop |
| `on-failure`     | Restart only if exit code ≠ 0          |
| `unless-stopped` | Restart unless manually stopped        |

Use this with healthchecks to keep your app resilient.

---

##  **Dependency Control Rules**

| Feature                        | Purpose                                                        |
| ------------------------------ | -------------------------------------------------------------- |
| **depends_on**                 | Define container startup order                                 |
| **condition: service_healthy** | Wait for healthcheck success before starting dependent service |
| **healthcheck**                | Periodic command that defines container readiness              |
| **restart policy**             | Auto-heal containers that fail or exit unexpectedly            |

---

##  **Lab 3 — Multi-Tier Example**

```yaml
version: "3.9"

services:
  redis:
    image: redis:alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      retries: 3

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: pass
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      retries: 5

  app:
    image: node:18
    command: sh -c "node server.js"
    depends_on:
      redis:
        condition: service_healthy
      db:
        condition: service_healthy
```

Now, both Redis and Postgres must be *healthy* before `app` starts — exactly how it should work in production.

---

##  **Key Takeaways**

| Concept                                     | Description                                            |
| ------------------------------------------- | ------------------------------------------------------ |
| **Healthcheck**                             | Ensures service readiness, not just startup            |
| **depends_on (condition: service_healthy)** | Enforces startup sequencing                            |
| **Restart policies**                        | Build resilience and auto-recovery                     |
| **Self-healing Stacks**                     | Combine these for reliable multi-service orchestration |

---

##  **Next Lesson → Lesson 24: Volumes & Persistence in Compose**

Next, we’ll learn how to share and persist data between containers in a Compose stack — from local volumes to databases that survive restarts.

