

#  **Lesson 17: Volume Lifecycle, Backup & Migration**

### *How to list, inspect, copy, backup, and move Docker volumes safely*

---

##  **Context**

In the previous lesson, you learned how to choose the right storage type — **Named Volumes, Bind Mounts, and tmpfs.**

Now, let’s take a real-world scenario:

> You’re running a production container with critical data (e.g., a MySQL database).
> The container can be recreated anytime — but your **volume data must never be lost.**

To manage this correctly, you must understand:

* How Docker **tracks, mounts, and detaches** volumes
* How to **backup, restore, or move** them across environments
* How to safely **inspect and clean** unused volumes

---

##  **Concept**

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/c17649c0-e337-4119-86a4-284a8a4e5039" />


A Docker **volume lifecycle** has these major stages:

```
CREATE → ATTACH → USE → DETACH → BACKUP → RESTORE → REMOVE
```

We’ll explore each one in detail with real examples.

---

##  **1️ Volume Creation**

Create a named volume:

```bash
docker volume create app_data
```

Inspect it:

```bash
docker volume inspect app_data
```

Output:

```json
{
  "Driver": "local",
  "Mountpoint": "/var/lib/docker/volumes/app_data/_data"
}
```

List all volumes:

```bash
docker volume ls
```

---

##  **2️ Attaching & Using Volumes**

Attach to a container:

```bash
docker run -d --name app1 -v app_data:/usr/share/nginx/html nginx
```

Data written inside `/usr/share/nginx/html` persists across container restarts.

To confirm:

```bash
docker exec app1 ls /usr/share/nginx/html
```

Even after removal:

```bash
docker rm -f app1
docker run --rm -v app_data:/data busybox ls /data
```

 Files are still there — proving persistence.

---

##  **3️ Detaching Volumes**

Volumes are **not deleted** when you remove a container.
To delete both:

```bash
docker rm -f app1 -v
```

 Use `-v` cautiously — it deletes attached volumes permanently.

---

##  **4️ Backing Up a Volume**

Volumes can be **archived** using another temporary container.

Example:
Backup the `app_data` volume to a tar file:

```bash
docker run --rm \
  -v app_data:/source \
  -v $(pwd):/backup \
  busybox tar czf /backup/app_data_backup.tar.gz -C /source .
```

This creates a compressed backup file in your current directory.

 Verify:

```bash
ls -lh app_data_backup.tar.gz
```

---

##  **5️ Restoring a Volume**

To restore from the backup file:

```bash
docker volume create app_data_restored

docker run --rm \
  -v app_data_restored:/target \
  -v $(pwd):/backup \
  busybox tar xzf /backup/app_data_backup.tar.gz -C /target
```

Inspect restored data:

```bash
docker run --rm -v app_data_restored:/data busybox ls /data
```

---

##  **6️ Migrating Volumes Between Hosts**

When you move workloads between servers (e.g., Dev → Prod),
you can migrate volumes by exporting the tarball and restoring it on another host.

**Steps:**

1. Copy backup file via `scp` or `rsync`

   ```bash
   scp app_data_backup.tar.gz user@target-server:/tmp/
   ```
2. On target server → restore using same command as above.

That’s all — your Docker data moves seamlessly between machines.

---

##  **7️ Cleaning Unused Volumes**

To list unused volumes:

```bash
docker volume ls -f dangling=true
```

To delete all unused volumes:

```bash
docker volume prune
```

 Confirm before pruning, especially on shared hosts.

---

##  **Hands-On Labs**

###  Lab 17.1 — Full Backup and Restore Demo

```bash
docker run -d --name dataapp -v myvol:/data busybox sh -c "echo 'dockAI rocks!' > /data/msg.txt"

# Backup
docker run --rm -v myvol:/source -v $(pwd):/backup busybox tar czf /backup/myvol.tar.gz -C /source .

# Remove container and volume
docker rm -f dataapp
docker volume rm myvol

# Restore
docker volume create myvol
docker run --rm -v myvol:/target -v $(pwd):/backup busybox tar xzf /backup/myvol.tar.gz -C /target
docker run --rm -v myvol:/data busybox cat /data/msg.txt
```

 Output → `dockAI rocks!`

---

###  Lab 17.2 — Volume Lifecycle Check

```bash
docker volume create tempvol
docker run --rm -v tempvol:/data busybox sh -c "echo 'Testing' > /data/test.txt"
docker volume inspect tempvol
docker volume ls
docker volume prune
```

You’ll clearly see how Docker tracks volume usage.

---

##  ** Tips**

*  Keep backups outside `/var/lib/docker` for safety.
*  Automate backups via cron using `busybox tar` jobs.
*  For production, use volume drivers like:

  * `local`
  * `nfs`
  * `aws-efs`
  * `azurefile`
  * `gcp-filestore`
*  Avoid bind mounts for production database storage.
*  Use `docker cp` for one-off file retrievals.

---

##  **Summary**

| Lifecycle Stage | Command / Concept        | Description                  |
| --------------- | ------------------------ | ---------------------------- |
| Create          | `docker volume create`   | Makes new persistent volume  |
| Attach          | `-v name:/path`          | Connects to container        |
| Use             | `docker exec` / app data | Active container writes data |
| Detach          | `docker rm`              | Volume stays intact          |
| Backup          | `busybox tar czf`        | Export data to host          |
| Restore         | `tar xzf`                | Recreate on new volume       |
| Clean           | `docker volume prune`    | Remove unused                |

---

##  **Next Lesson → Lesson 18: Exploring Advanced Volume Drivers**

We’ll explore **NFS, Azure File, AWS EFS, and GCP Filestore integrations**,
showing how Docker extends beyond local disks — into **multi-host and cloud storage**.

---

