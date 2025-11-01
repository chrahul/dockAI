

#  **Lesson 16: Named Volumes vs Bind Mounts vs tmpfs**

### *Choosing the right storage type for durability, performance, and security*

---

##  **Context**

In Lesson 15, we explored how Docker layers form the **ephemeral container filesystem** using `overlay2`.
Now we’ll study how to **extend container life** beyond the writable layer using *persistent storage mechanisms.*

Every real-world application — databases, web servers, or APIs — needs some way to **save data** even after a container stops or restarts.

Docker gives you **three main options** for persistence:

1. **Named Volumes** → Docker-managed storage inside `/var/lib/docker/volumes`
2. **Bind Mounts** → Host-managed directories mapped directly
3. **tmpfs Mounts** → Memory-backed filesystems for speed and security

Let’s explore each — conceptually, practically, and strategically.

---

##  **Concept**

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/c2355d0b-9fbb-4b90-a180-39c74dae8722" />



### 1️ **Named Volumes – Docker Managed**

####  What It Is

A **named volume** is storage created and managed by Docker itself.
You don’t worry about exact file paths — Docker keeps it under its managed directory.

####  Default Path

```
/var/lib/docker/volumes/<volume_name>/_data
```

####  Create Volume

```bash
docker volume create app_data
```

####  Use Volume in Container

```bash
docker run -d --name app1 -v app_data:/usr/share/nginx/html nginx
```

Even if you remove the container:

```bash
docker rm -f app1
```

the data inside `/usr/share/nginx/html` persists because it’s stored in `app_data`.

####  Inspect Volume

```bash
docker volume inspect app_data
```

Output:

```json
{
  "Mountpoint": "/var/lib/docker/volumes/app_data/_data",
  "Driver": "local"
}
```

 Best for:

* Databases (MySQL, PostgreSQL, MongoDB)
* Persistent application state
* Backup and migration (Docker-managed lifecycle)

---

### 2️ **Bind Mounts – Host Managed**

####  What It Is

A **bind mount** links a directory or file from your **host machine** directly into the container.
It’s instant, flexible, and ideal for development environments.

####  Example

```bash
mkdir ~/dockai_site
echo "Hello DockAI!" > ~/dockai_site/index.html

docker run -d -p 8080:80 \
  -v ~/dockai_site:/usr/share/nginx/html \
  nginx
```

Any change in your host folder updates instantly inside the container.

####  Host Path Example

```
Host:      /home/rahul/dockai_site
Container: /usr/share/nginx/html
```

 Best for:

* Development environments
* Local testing and file sync
* Custom configurations

 Not recommended for production because:

* Depends on host path stability
* Full host access — can overwrite system files

---

### 3️ **tmpfs Mounts – In-Memory**

#### 🔹 What It Is

`tmpfs` mounts create temporary storage **in memory** (RAM), not on disk.
Fast and secure — but lost when the container stops.

####  Example

```bash
docker run -d \
  --name tempapp \
  --mount type=tmpfs,destination=/app/tmp \
  busybox sleep 1000
```

Check:

```bash
docker exec tempapp df -h /app/tmp
```

Output:

```
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           1.9G     0  1.9G   0% /app/tmp
```

 Best for:

* Caching
* Session data
* Sensitive temporary files
* High-speed read/write operations

---

##  **Practical Comparison**

| Feature         | **Named Volume**              | **Bind Mount**     | **tmpfs Mount**     |
| --------------- | ----------------------------- | ------------------ | ------------------- |
| **Managed By**  | Docker                        | User/Host          | Kernel (RAM)        |
| **Location**    | `/var/lib/docker/volumes/...` | Host path          | In-memory           |
| **Persistence** | Yes                             | Yes                 | (temporary)       |
| **Performance** | High                          | Depends on host FS | Very high (RAM)     |
| **Isolation**   | High                          | Low                | High                |
| **Use Case**    | Databases, prod data          | Dev sync, config   | Temp cache, secrets |

---

##  **Labs – Step by Step**

### Lab 16.1 – Named Volume Persistence

```bash
docker run -d --name voltest -v appdata:/data busybox sh -c "echo 'dockAI Volume Test' > /data/file.txt"
docker rm -f voltest
docker run -d --name voltest2 -v appdata:/data busybox cat /data/file.txt
```

 Output → `dockAI Volume Test` (data persisted)

---

### Lab 16.2 – Bind Mount Sync

```bash
mkdir ~/dockai_bind
echo "Sync from Host" > ~/dockai_bind/info.txt

docker run --rm -v ~/dockai_bind:/data busybox cat /data/info.txt
```

Edit host file and rerun — change reflects instantly inside the container.

---

### Lab 16.3 – tmpfs Speed Test

```bash
docker run --rm --mount type=tmpfs,destination=/cache busybox sh -c "dd if=/dev/zero of=/cache/test bs=1M count=100"
```

Observe ultra-fast write — stored in RAM, gone when container stops.

---

##  **Notes**

* Use **named volumes** for production data — portable, managed, easy to back up.
* Use **bind mounts** for dev — editable code sync, quick testing.
* Use **tmpfs** when performance or privacy matters (e.g., encryption keys).
* You can mix multiple mount types in the same container.
* Use `docker inspect <container>` → `.Mounts` to verify configuration.

---

##  **Summary**

| Scenario                   | Recommended Storage Type |
| -------------------------- | ------------------------ |
| Database or App Data       | **Named Volume**         |
| Local Development          | **Bind Mount**           |
| High-Speed Temporary Cache | **tmpfs**                |

---

##  **Next Lesson → Lesson 17: Volume Lifecycle, Backup & Migration**

We’ll explore how to **list, inspect, copy, backup, and restore** Docker volumes safely — including techniques for **moving volumes between servers** and **automating backups** with containers.

---

