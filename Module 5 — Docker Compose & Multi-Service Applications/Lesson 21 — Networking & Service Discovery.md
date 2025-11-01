

#  **Lesson 21 — Networking & Service Discovery**

### *How containers talk to each other inside Docker Compose*

---

##  **Context**

In Module 3, we explored Docker’s native networking — bridge networks, veth pairs, and the docker0 interface.
Now, within **Docker Compose**, networking becomes automatic and more intelligent.

When you launch a Compose stack, Docker automatically creates:

* A **dedicated bridge network** for your project, and
* A **DNS-based service discovery system** that lets containers communicate **by name**, not by IP.

This eliminates hardcoded addresses and allows your services to find each other dynamically.

> “In Docker Compose, containers don’t need IPs — they just call each other by name.”

---

##  **Concept: Default Networking in Compose**


<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/6d69acd5-7b2a-4197-a3df-3e4eb00dc7b7" />


By default, every Compose project gets:

* **One isolated network** named `<project>_default`
* Every **service** in your `docker-compose.yml` joins that network
* Containers can resolve each other via **built-in DNS**

Example:

```yaml
version: "3.9"

services:
  web:
    image: nginx
    ports:
      - "8080:80"
  app:
    image: python:3.10
    command: python app.py
    depends_on:
      - db
  db:
    image: postgres:15
```

Here, `app` can reach the database using hostname `db`:

```python
psycopg2.connect(host="db", user="postgres", password="secret")
```

You never need to know the IP address — Docker handles DNS internally.

---

##  **Networking Types in Docker Compose**

| Network Type         | Description                                      | Use Case                             |
| -------------------- | ------------------------------------------------ | ------------------------------------ |
| **Bridge (default)** | Isolated network between containers on same host | Development / local environments     |
| **Host**             | Shares host network namespace                    | Performance testing or special cases |
| **None**             | No networking at all                             | Isolated jobs or secure sandbox      |
| **External**         | Reuse an existing Docker network                 | Connect multiple stacks together     |

---

##  **Lab 1 — Observe the Default Network**

### Step 1: Create a simple stack

```bash
mkdir net-demo && cd net-demo
nano docker-compose.yml
```

Paste:

```yaml
version: "3.9"
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
  app:
    image: alpine
    command: ping web
```

### Step 2: Launch the stack

```bash
docker compose up
```

You’ll see output like:

```
app_1  | PING web (172.19.0.2): 56 data bytes
app_1  | 64 bytes from 172.19.0.2: seq=0 ttl=64 time=0.086 ms
```

 The `app` container resolved `web` through Compose’s internal DNS.

---

##  **Lab 2 — Custom Network and Isolation**

You can create **multiple custom networks** for fine-grained isolation.

```yaml
version: "3.9"

services:
  frontend:
    image: nginx
    ports:
      - "8080:80"
    networks:
      - public_net

  backend:
    image: python:3.10
    command: python -m http.server 5000
    networks:
      - private_net

networks:
  public_net:
  private_net:
```

Now:

* `frontend` cannot reach `backend`
* Containers in the same network (e.g., `public_net`) can talk freely

Check networks:

```bash
docker network ls
docker network inspect <project>_public_net
```

---

##  **Lab 3 — Linking Two Stacks via External Network**

You can even connect **two separate Compose projects** together:

1. Create a shared network:

   ```bash
   docker network create shared_net
   ```

2. In Stack A:

   ```yaml
   networks:
     default:
       external: true
       name: shared_net
   ```

3. In Stack B:

   ```yaml
   networks:
     default:
       external: true
       name: shared_net
   ```

Now both stacks can discover each other by service name across projects.

---

## **Inspect and Debug Networking**

Common commands:

```bash
docker network ls                # List networks
docker network inspect <name>    # Inspect configuration
docker exec -it <container> sh   # Ping or curl other containers
docker compose logs <service>    # Check connectivity errors
```

---

##  **Key Takeaways**

| Concept               | Description                                          |
| --------------------- | ---------------------------------------------------- |
| **Service Discovery** | Containers can resolve each other using built-in DNS |
| **Default Network**   | Every stack gets its own isolated bridge network     |
| **Custom Networks**   | Allow grouping or isolation of services              |
| **External Networks** | Enable cross-stack communication                     |
| **Ports Mapping**     | `8080:80` publishes container ports to host          |

---

##  **Next Lesson → Lesson 22: Environment Management & Profiles**

We’ll learn how to manage **environment variables**, **.env files**, and **profiles** to create **dev, test, and prod** configurations from a single Compose file.

---

