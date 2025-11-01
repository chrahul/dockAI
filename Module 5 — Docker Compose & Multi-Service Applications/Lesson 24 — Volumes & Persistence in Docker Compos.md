
#  **Lesson 24 — Volumes & Persistence in Docker Compose**

### *Designing data that survives restarts and flows between services*

---

## **Context**

In **Module 4**, we explored how Docker stores data — using **overlay2**, **bind mounts**, and **volumes**.
Now, inside **Docker Compose**, these same primitives are managed declaratively.

With Compose, you can define:

* Which containers share data
* Which volumes persist across stack restarts
* How data moves between **services**, **networks**, and **hosts**

> “Stateless services scale fast.
> But persistent services build trust.”

---

##  **Concept: Persistence in Multi-Service Stacks**


<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/3442ca7d-967f-4595-8b0d-b01727b781a8" />


When you shut down your stack with `docker compose down`, containers are destroyed —
but volumes can remain intact (unless removed manually).

### **Types of Persistence in Compose**

| Type                 | Description                        | Example Use                |
| -------------------- | ---------------------------------- | -------------------------- |
| **Named Volume**     | Managed by Docker                  | Databases, cache storage   |
| **Bind Mount**       | Host path mapped to container path | Local dev files            |
| **Anonymous Volume** | Auto-created, unnamed              | Temporary scratch data     |
| **tmpfs**            | In-memory storage                  | High-speed ephemeral cache |

---

##  **Example: Database with Named Volume**

```yaml
version: "3.9"
services:
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

### Key points:

* The volume `db_data` persists even if the container stops.
* Run `docker volume ls` to verify persistence.

---

##  **Lab 1 — Persist MySQL Data**

### Step 1: Create and run

```bash
docker compose up -d
```

### Step 2: Connect and insert data

```bash
docker exec -it <container_id> mysql -u root -p
mysql> CREATE TABLE demo (id INT PRIMARY KEY);
mysql> INSERT INTO demo VALUES (1);
```

### Step 3: Tear down

```bash
docker compose down
```

### Step 4: Restart

```bash
docker compose up -d
```

 Data remains! Because the named volume `db_data` persists under `/var/lib/docker/volumes`.

---

##  **Example: Shared Volume Between Services**

You can let multiple containers share the same storage area.

```yaml
version: "3.9"
services:
  app:
    image: busybox
    command: sh -c "echo 'Hello from app' >> /shared/logs.txt && sleep 3600"
    volumes:
      - shared_data:/shared

  logger:
    image: busybox
    command: sh -c "tail -f /shared/logs.txt"
    volumes:
      - shared_data:/shared

volumes:
  shared_data:
```

 Both containers share `/shared`, writing and reading the same file.
A perfect mini example of **sidecar-style persistence**.

---

##  **Lab 2 — Bind Mounts for Local Development**

For developers, it’s useful to sync local code instantly.

```yaml
version: "3.9"
services:
  web:
    image: nginx:alpine
    volumes:
      - ./html:/usr/share/nginx/html
    ports:
      - "8080:80"
```

* Modify files inside `html/` on your laptop
* Browser auto-refreshes with changes

 Ideal for frontend or Node.js workflows.

---

##  **Lab 3 — Volume Backup and Restore in Compose**

You can back up data volumes defined in Compose:

```bash
docker run --rm \
  -v myproject_db_data:/data \
  -v $(pwd):/backup \
  busybox tar czf /backup/db_backup.tar.gz /data
```

Restore it later:

```bash
docker run --rm \
  -v myproject_db_data:/data \
  -v $(pwd):/backup \
  busybox tar xzf /backup/db_backup.tar.gz -C /data
```

---

##  **Pro Tips for Compose Volumes**

| Practice                                          | Why It Matters                        |
| ------------------------------------------------- | ------------------------------------- |
| Use **named volumes** for all persistent services | Stable, reusable storage              |
| Never mount `/var/lib/docker`                     | Avoid corrupting Docker internals     |
| Bind mounts for local dev only                    | Prevent permission issues in prod     |
| Version your **Compose volume names**             | e.g., `db_data_v2` to avoid conflicts |
| Always use **separate volumes per service**       | Isolation improves portability        |

---

##  **Key Takeaways**

| Concept            | Description                     |
| ------------------ | ------------------------------- |
| **Named Volumes**  | Persistent, Docker-managed data |
| **Bind Mounts**    | Host-to-container live sync     |
| **Shared Volumes** | Data sharing between services   |
| **tmpfs**          | Memory-backed temporary storage |
| **Backup/Restore** | Protect and migrate data safely |

---

##  **Next Lesson → Lesson 25: Scaling & Multi-Container Patterns**

We’ll explore **scaling strategies**, **load balancing**, and **high availability patterns** — learning how to scale specific services with `--scale` and design stateless architectures that can handle real-world traffic.

