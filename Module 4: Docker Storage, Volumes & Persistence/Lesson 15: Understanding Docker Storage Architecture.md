

#  **Lesson 15: Understanding Docker Storage Architecture (overlay2 + Mounts)**

### *Where your container’s data actually lives and how Docker glues it together*

---

##  **Context**

Every container looks like a tiny machine with its own filesystem —
`/bin`, `/etc`, `/usr`, `/var` — everything seems real.
But underneath, Docker isn’t creating an actual disk; it’s assembling **multiple read-only layers plus a writable one** using Linux’s **OverlayFS (overlay2)**.

Understanding this mechanism is essential to:

* Debug why your container “lost” data after restart,
* Optimize image sizes and build caching, and
* Design proper persistence strategies using volumes.

Real-world example:
When you run a **MySQL container**, the database stores data under `/var/lib/mysql`.
If that directory sits on the container’s *ephemeral* writable layer, all data is gone once the container is deleted.
That’s why we use **volumes** — to move it outside overlay2 and make it persistent.

---

##  **Concept**

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/cc997577-0aad-4480-9261-afbb7d03d267" />


### 1️ Docker’s File System Stack

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/a9ab21d9-3207-46af-bcbb-b2af1c854ff8" />

```
Container FS
 ├── Writable Layer  (unique to each container)
 ├── Read-only Layers (from image)
 ├── Union Mount (overlay2 merges them)
 └── Volumes / Mounts (optional, for persistence)
```

When you inspect `/var/lib/docker/overlay2`, you’ll see folders named like:

```
/var/lib/docker/overlay2/
 ├── 7e2bcd.../
 │   ├── diff/     ← contents of this image layer
 │   ├── lower/    ← links to lower layers
 │   ├── merged/   ← union mount seen by container
 │   ├── work/
```

### 2️ The Overlay2 Mechanism

OverlayFS merges directories from different layers:

```
upperdir  →  container’s writable layer
lowerdir  →  image’s read-only layers
workdir   →  internal workspace for fs ops
merged    →  final unified view inside container
```

You can inspect this mapping:

```bash
sudo mount | grep overlay
```

Sample output:

```
overlay on /var/lib/docker/overlay2/7e2bcd.../merged
 type overlay (rw, lowerdir=.../lower,upperdir=.../diff,workdir=.../work)
```

That line is the **truth** of container storage.

---

### 3️ Ephemeral vs Persistent Data

| Type         | Location                            | Survives `docker rm`? | Use Case               |
| ------------ | ----------------------------------- | --------------------- | ---------------------- |
| Ephemeral    | `/var/lib/docker/overlay2/.../diff` | No                     | App runtime data, logs |
| Named Volume | `/var/lib/docker/volumes/...`       | Yes                   | Databases, uploads     |
| Bind Mount   | Host directory you specify          | Yes                     | Sharing code, configs  |
| tmpfs        | In-memory only                      | until stop        | Fast transient storage |

---

### 4️ Inspecting Mounts

```bash
docker inspect <container_name> --format '{{ json .Mounts }}' | jq
```

Output:

```json
[
  {
    "Type": "volume",
    "Name": "mydata",
    "Destination": "/data",
    "Driver": "local",
    "Mode": "",
    "RW": true,
    "Propagation": ""
  }
]
```

---

##  **Labs – Step by Step**

###  Lab 15.1 – Explore overlay2 Directory

1. **Run an Nginx container**

   ```bash
   docker run -d --name web1 nginx
   ```

2. **Find its mount path**

   ```bash
   docker inspect web1 --format '{{ .GraphDriver.Data.MergedDir }}'
   ```

   Example:

   ```
   /var/lib/docker/overlay2/7e2bcd.../merged
   ```

3. **List files from host**

   ```bash
   sudo ls /var/lib/docker/overlay2/7e2bcd.../merged/etc/nginx/
   ```

    You’re now looking *inside* the container filesystem from the host!

---

###  Lab 15.2 – Demonstrate Ephemeral Data Loss

1. Create a file inside the container:

   ```bash
   docker exec web1 touch /tmp/demo.txt
   docker exec web1 ls /tmp/
   ```
2. Remove and recreate the container:

   ```bash
   docker rm -f web1
   docker run -d --name web1 nginx
   docker exec web1 ls /tmp/
   ```

    `demo.txt` is gone — because it was inside the writable layer.

---

###  Lab 15.3 – Attach a Named Volume

1. Run container with volume:

   ```bash
   docker run -d --name web2 -v myvol:/data nginx
   ```
2. Create file:

   ```bash
   docker exec web2 bash -c "echo 'Hello DockAI' > /data/hello.txt"
   ```
3. Remove container and re-attach volume:

   ```bash
   docker rm -f web2
   docker run -d --name web3 -v myvol:/data nginx
   docker exec web3 cat /data/hello.txt
   ```

    Data persists!

---

###  Lab 15.4 – Bind Mount from Host

```bash
mkdir ~/dockai_data
echo "Persistent content" > ~/dockai_data/demo.txt

docker run -d --name web4 -v ~/dockai_data:/usr/share/nginx/html nginx
```

Access via browser:
 `http://localhost:8080/demo.txt`

Any change in `~/dockai_data` reflects instantly.

---

##  **Notes**

* overlay2 replaced AUFS — faster and simpler.
* Each layer is immutable; new containers reuse existing ones.
* Inspect `docker system df -v` to see space used by images and containers.
* Keep `/var/lib/docker` on a large, fast disk (SSD if possible).
* Never delete overlay2 manually — always through Docker commands.

---

##  **Summary**

| Component          | Description                          |
| ------------------ | ------------------------------------ |
| **overlay2**       | Merges image + container layers      |
| **Writable Layer** | Ephemeral per-container layer        |
| **Volume**         | Persistent storage managed by Docker |
| **Bind Mount**     | Direct host directory mapping        |
| **tmpfs**          | Memory-only filesystem               |




The **most misunderstood topics** in Docker: Volumes vs Bind Mounts
Let’s break it down:

---

#  **Volumes vs Bind Mounts — The Complete Difference**

When you want to **store data persistently** in Docker, you have two main options:

* **Named Volumes** (managed by Docker)
* **Bind Mounts** (direct mapping from your host)

Let’s compare them step by step 

---

##  1️ The Core Difference

| Feature              | **Named Volume**                                        | **Bind Mount**                                   |
| -------------------- | ------------------------------------------------------- | ------------------------------------------------ |
| **Who creates it?**  | Docker manages it automatically                         | You provide a host path manually                 |
| **Storage Location** | `/var/lib/docker/volumes/...`                           | Anywhere on your host (e.g., `/home/rahul/data`) |
| **Persistence**      | Persists even if the container is deleted               | Persists as long as host path exists             |
| **Use Case**         | Databases, persistent app data                          | Code sharing, configuration files                |
| **Portability**      | Easily moved/exported with `docker volume`              | Tied to specific host directory                  |
| **Performance**      | Optimized by Docker                                     | Depends on host filesystem                       |
| **Security**         | Isolated, managed by Docker                             | Full host access (less isolation)                |
| **Backup / Restore** | `docker volume inspect`, `docker run --rm -v vol:/data` | Standard file copy on host                       |
| **Ownership**        | Controlled by container runtime                         | Controlled by host user permissions              |

---

##  2️ Real-World Example

### Named Volume (Docker-managed)

```bash
docker run -d \
  --name mysql1 \
  -v mysql_data:/var/lib/mysql \
  mysql:8
```

Docker automatically creates:

```
/var/lib/docker/volumes/mysql_data/_data/
```

Easy to manage
Survives container removal
Ideal for DB storage, uploaded files, etc.

Inspect it:

```bash
docker volume inspect mysql_data
```

Output:

```json
{
  "Mountpoint": "/var/lib/docker/volumes/mysql_data/_data",
  "Driver": "local"
}
```

---

### Bind Mount (Host Path)

```bash
docker run -d \
  --name web1 \
  -v /home/rahul/website:/usr/share/nginx/html \
  nginx
```

Docker maps *your host directory* directly into the container:

* `/home/rahul/website` ↔ `/usr/share/nginx/html`

Now, changes in either place reflect instantly:

```bash
echo "Welcome!" > /home/rahul/website/index.html
```

→ Instantly available inside the container.

Great for **local development**
Risky for production (can modify host files unintentionally)

---

##  3️ Visual Summary

```
              +--------------------------+
              |     Container Filesystem  |
              |---------------------------|
              | /usr/share/nginx/html     | → ←  Bind Mount (Host Path)
              | /var/lib/mysql            | → ←  Named Volume (Managed by Docker)
              +---------------------------+
```

* **Named Volume** → Docker manages path internally.
* **Bind Mount** → You directly mount a folder from host.

---

##  4️ Quick Labs

### Lab A – Check Volume Location

```bash
docker volume create myvol
docker run -v myvol:/data busybox sh -c "echo hello > /data/file.txt"
sudo cat /var/lib/docker/volumes/myvol/_data/file.txt
```

Output → `hello`

---

### Lab B – Check Bind Mount Reflection

```bash
mkdir ~/dockai_data
echo "dockAI rocks!" > ~/dockai_data/notes.txt

docker run -v ~/dockai_data:/data busybox cat /data/notes.txt
```

Output → `dockAI rocks!`

Edit file on host → instant update inside container 

---

##  5️ Pro Tips

* **Named Volumes** are safer for **production data** — Docker handles lifecycle.
* **Bind Mounts** are ideal for **local dev** where you need live file sync.
* Use `docker volume ls` and `docker volume inspect` for managed volumes.
* Avoid mounting `/var/lib/docker` or system paths.
* In Docker Compose, always prefer **volumes:** section for persistence.

---

##  6️ Analogy

Think of it like this 

| Type             | Analogy                                                                                                                                   |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| **Named Volume** | Like renting a safe locker from Docker — you don’t know its exact location, but Docker keeps it secure and gives you a key (volume name). |
| **Bind Mount**   | Like opening your home drawer to Docker — quick access, but you’re responsible for what happens inside.                                   |

---
### Concept Recap

We’ll show:

* On the **left:** *Named Volume* (Docker-managed, inside `/var/lib/docker/volumes/...`)
* On the **right:** *Bind Mount* (host-managed, from `/home/rahul/...`)
* Both connected to the container’s `/data` folder
* Clean dark navy background, dockAI color palette (blue/orange), flat tech look

---





---


##  **Next Lesson → Lesson 16: Named Volumes vs Bind Mounts vs tmpfs**

We’ll go deeper into **how to choose the right storage type**, understand **mount propagation**, and design persistent storage for **stateful microservices like MySQL and Postgres**.

---

