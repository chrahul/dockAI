

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

---

##  **Next Lesson → Lesson 16: Named Volumes vs Bind Mounts vs tmpfs**

We’ll go deeper into **how to choose the right storage type**, understand **mount propagation**, and design persistent storage for **stateful microservices like MySQL and Postgres**.

---

