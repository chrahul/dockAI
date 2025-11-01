

#  **Lesson 22 — Environment Management & Profiles**

### *Managing Configurations Across Dev, Test, and Prod with Docker Compose*

---

##  **Context**

When working on modern applications, you often face this challenge:

> “How do I use the same Docker Compose file for development, testing, and production —
> without duplicating everything?”

Docker Compose solves this elegantly with:

1. **Environment variables**
2. **.env files**
3. **Profiles**

These features make your setup flexible, portable, and CI/CD-friendly.

---

##  **Concept: Environment Variables in Compose**


<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/c87b5952-7fa3-4f5b-b60a-d1d94096149e" />


Environment variables help you **parametrize your setup** — so you can define values (like passwords, ports, or image tags) once and reuse them everywhere.

You can pass environment variables in three main ways:

| Method                           | Description                          | Example                       |
| -------------------------------- | ------------------------------------ | ----------------------------- |
| **Inline in docker-compose.yml** | Define directly under a service      | `environment:`                |
| **From a .env file**             | Load automatically when Compose runs | `.env` in same folder         |
| **From the shell**               | Pass dynamically at runtime          | `VAR=value docker compose up` |

---

###  Example 1 — Inline Environment Variables

```yaml
version: "3.9"
services:
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: appdb
```

---

###  Example 2 — Using a `.env` File

`.env` file:

```
POSTGRES_USER=admin
POSTGRES_PASSWORD=supersecret
POSTGRES_DB=appdb
```

`docker-compose.yml`:

```yaml
version: "3.9"
services:
  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
```

Now, simply run:

```bash
docker compose up -d
```

 Docker Compose automatically reads `.env` and substitutes the values.

---

###  Example 3 — Overriding Variables from Shell

```bash
POSTGRES_PASSWORD=overridepass docker compose up -d
```

This overrides `.env` values for the current session only — useful for CI/CD pipelines.

---

##  **Lab 1 — Multi-Environment Setup**

Let’s create a setup that can switch between dev and prod environments seamlessly.

### Step 1: Directory structure

```
compose-env-demo/
 ├── docker-compose.yml
 ├── .env.dev
 └── .env.prod
```

### Step 2: docker-compose.yml

```yaml
version: "3.9"

services:
  web:
    image: nginx:${NGINX_TAG}
    ports:
      - "${HOST_PORT}:80"
```

### Step 3: .env.dev

```
NGINX_TAG=alpine
HOST_PORT=8080
```

### Step 4: .env.prod

```
NGINX_TAG=latest
HOST_PORT=80
```

### Step 5: Run with dev config

```bash
cp .env.dev .env
docker compose up -d
```

Switch to production:

```bash
cp .env.prod .env
docker compose up -d
```

 The same Compose file behaves differently based on environment variables — no duplication!

---

##  **Profiles: Activate Services on Demand**

Profiles let you **conditionally enable or disable services** inside your Compose file.

This is extremely useful for:

* Separating **dev-only tools** (like adminer or pgAdmin)
* Enabling **optional components** (like monitoring)
* Keeping **production clean**

---

###  Example — Using Profiles

```yaml
version: "3.9"

services:
  web:
    image: nginx
    ports:
      - "8080:80"

  db:
    image: postgres:15

  adminer:
    image: adminer
    ports:
      - "8081:8080"
    profiles:
      - debug
```

### Run with profile:

```bash
docker compose --profile debug up -d
```

 Only `web`, `db`, and `adminer` (under profile `debug`) will start.

Without profile:

```bash
docker compose up -d
```

 `adminer` won’t start.

---

##  **Lab 2 — Combine Profiles + .env**

Combine `.env` and profiles for flexible setups:

`.env.dev`

```
ENABLE_DEBUG=true
```

`docker-compose.yml`

```yaml
version: "3.9"

services:
  app:
    image: myapp:${TAG}
    ports:
      - "8080:80"

  debug:
    image: adminer
    ports:
      - "8081:8080"
    profiles:
      - debug
```

Conditional run:

```bash
if [ "$ENABLE_DEBUG" = "true" ]; then
  docker compose --profile debug up -d
else
  docker compose up -d
fi
```

---

##  **Key Takeaways**

| Feature                   | Purpose                       | Benefit                                 |
| ------------------------- | ----------------------------- | --------------------------------------- |
| **Environment Variables** | Dynamic configuration         | Keep Compose portable                   |
| **.env Files**            | Store config securely         | Simplify switching between environments |
| **Profiles**              | Conditional service inclusion | Control what runs per environment       |
| **Runtime Overrides**     | Temporary variable changes    | Great for CI/CD flexibility             |

---

##  **Next Lesson → Lesson 23: Healthchecks & Dependency Control**

We’ll dive into **service health**, startup sequencing, and dependency management — ensuring containers start in the right order and self-heal when needed.

