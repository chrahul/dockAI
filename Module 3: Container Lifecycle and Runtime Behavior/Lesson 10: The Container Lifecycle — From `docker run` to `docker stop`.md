
#  Lesson 10: The Container Lifecycle — From `docker run` to `docker stop`

### *(Module 3 – Container Lifecycle & Runtime Behavior)*

---

##  Context

So far, you’ve learned *how* Docker **builds** images.
Now it’s time to see **how those images come alive as containers.**

When you execute:

```bash
docker run nginx
```

a *lot* happens under the hood — image checks, filesystem mounts, namespaces, cgroups, networking, process spawning, logging hooks, and more.

This lesson breaks that invisible sequence into a clear, traceable lifecycle — from **creation → execution → stop → removal**.

---

##  Learning Goals

 Understand each Docker container state and its transitions
 Learn what really happens when you execute `docker run`
 Differentiate between `docker create`, `start`, `pause`, `restart`, and `stop`
 Use CLI commands to explore container metadata and runtime behavior
 Trace events using `docker events`

---

##  Concept

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/9241c1f1-3d8a-4182-b045-5e150c11e582" />


### 1️ Container States

| State                | Description                                | Command Examples      |
| -------------------- | ------------------------------------------ | --------------------- |
| **Created**          | Filesystem prepared, container not started | `docker create nginx` |
| **Running**          | Main process (PID 1) executing             | `docker start nginx`  |
| **Paused**           | All processes frozen via cgroups freezer   | `docker pause nginx`  |
| **Stopped (Exited)** | Process ended, filesystem still exists     | `docker stop nginx`   |
| **Removed**          | All container data deleted                 | `docker rm nginx`     |

---

### 2️ The `docker run` Flow

When you type:

```bash
docker run -d -p 8080:80 nginx
```

<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/e2105753-a040-4c22-9286-39f446a2915d" />


Here’s what really happens step by step 

1. **CLI → Docker API** – `docker run` sends a `POST /v1.44/containers/create` and `/start` call to the Docker Daemon.
2. **Image Check → Pull if Missing** – daemon verifies `nginx:latest` locally, pulls from Docker Hub if needed.
3. **Filesystem Layer Mount** – Union FS (`overlay2`) creates a read-write layer on top of image layers.
4. **Container Namespace Setup** – Linux namespaces (PID, NET, MNT, UTS, IPC, USER) are created.
5. **Network Allocation** – A veth pair is attached to `docker0` bridge (or custom network).
6. **Cgroup Limits** – CPU/memory/pid quotas applied via cgroups v2.
7. **Process Spawn** – `/docker-entrypoint.sh nginx -g daemon off;` runs as PID 1 inside namespace.
8. **Logging Pipeline Attached** – stdout/stderr redirected to Docker’s log driver.
9. **Healthcheck (optional)** – If configured, health probe starts in background.

That one command orchestrates networking, storage, security and process isolation — within milliseconds.

---

### 3️ Lifecycle Commands and Transitions

| Action                | Transition                  | What Happens Internally                       |
| --------------------- | --------------------------- | --------------------------------------------- |
| `docker start <id>`   | created → running           | Re-attaches cgroups and starts PID 1          |
| `docker pause <id>`   | running → paused            | Freezes processes (via cgroups freezer)       |
| `docker unpause <id>` | paused → running            | Unfreezes processes                           |
| `docker stop <id>`    | running → stopped           | Sends `SIGTERM`, then `SIGKILL` after timeout |
| `docker restart <id>` | running → stopped → running | Gracefully restarts the process               |
| `docker rm <id>`      | stopped → removed           | Deletes filesystem layers and metadata        |

---

##  Labs — Step by Step

### Lab 10.1 — Create and Inspect Lifecycle

```bash
docker pull nginx:latest
docker create --name web1 -p 8080:80 nginx
docker ps -a
```

**Expected Output**

```
CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES
a9b3f4   nginx   "/docker-entrypoint.sh…"   2 seconds ago   Created   web1
```

---

### Lab 10.2 — Start and Explore

```bash
docker start web1
docker exec -it web1 bash
ps -ef
```

**Output Highlights**

```
PID   CMD
1   nginx: master process nginx -g daemon off;
7   nginx: worker process
```

→ PID 1 inside container is the master process — critical for signal handling.

---

### Lab 10.3 — Pause and Unpause

```bash
docker pause web1
docker stats --no-stream
docker unpause web1
```

While paused, CPU and memory usage freeze to 0.

---

### Lab 10.4 — Stop and Observe Signals

```bash
docker stop web1
docker inspect web1 --format '{{.State}}'
```

**Output**

```
{ "Status":"exited","ExitCode":0,"FinishedAt":"..."}
```

`docker stop` first sends SIGTERM, waits 10 s, then SIGKILL if not terminated.

---

### Lab 10.5 — Event Tracing

Run this in one terminal:

```bash
docker events
```

In another:

```bash
docker start web1 && docker stop web1 && docker rm web1
```

**Sample Output**

```
container create a9b3f4 (nginx)
container start a9b3f4
container stop a9b3f4
container destroy a9b3f4
```

 This is the exact event timeline of a container lifecycle.

---

##   Notes

* `docker run` = `docker create + docker start`.
* PID 1 handles all signals inside containers; mis-handling causes zombie processes.
* `docker ps -a` shows status of all states.
* For automation, use `docker events --since 10m --filter event=start` to trigger workflows.
* Lifecycle understanding is the foundation for Kubernetes Pods (coming later).

---

##  Summary

* Docker containers progress through well-defined states.
* `docker run` is not a single action but a sequence of daemon calls.
* Namespaces + cgroups create isolation at runtime.
* Logs and events let you trace the full container journey.

---

###  Next Lesson → **Lesson 11: The PID 1 Problem & Process Management**


---

Would you like me to generate a **dockAI-style diagram** for this lesson — a **solid flow visual** showing the container state machine (`create → start → pause → stop → remove`) with event arrows and icons?
