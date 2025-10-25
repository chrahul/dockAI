
#  Lesson 11: The PID 1 Problem & Process Management

### *(Module 3 – Container Lifecycle & Runtime Behavior)*

---

##  Context

Every container has its own isolated process tree.
When you start a container, Docker launches your application **as PID 1** inside its namespace.

For example:

```bash
docker run --name app alpine sleep 1000
```

Inside that container:

```bash
ps -ef
```

Output →

```
PID   CMD
1     sleep 1000
```

Simple, right?
But this PID 1 behaves *differently* from PID 1 on your host Linux.
And this difference causes some of the strangest bugs in production — zombie processes, stuck signals, and unresponsive containers.

---

##  Learning Goals

 Understand why PID 1 has special signal-handling behavior
 Learn what zombie processes are and how they appear in containers
 Discover how to use **tini** or **init wrappers** to fix them
 Experiment with process hierarchies and signal forwarding

---

##  Concept

<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/5d453d05-8bb2-4857-b94d-a25c96e7b1e1" />


### 1️ How Linux Processes Work

Every process in Linux is spawned by another (its parent).
The very first process — **PID 1 (init/systemd)** — starts everything else.
When a process dies, it sends a `SIGCHLD` signal to its parent so that the parent can **reap** (clean up) its zombie child.

If the parent doesn’t reap it, the process becomes a **zombie** — consuming a PID entry in the process table.

---

### 2️ Inside a Docker Container

When Docker starts a container, it launches your app as **PID 1** in that namespace.
There’s no full init system unless you explicitly run one.

Hence → your app is responsible for:

* Handling signals (SIGTERM, SIGINT)
* Reaping child processes (created via forks)

Most apps don’t do this — they expect systemd or init to handle it.
This leads to two classic problems:

| Issue                | Description                                                                    |
| -------------------- | ------------------------------------------------------------------------------ |
| **Zombie Processes** | Orphaned children not reaped by PID 1                                          |
| **Signal Loss**      | Container doesn’t exit when stopped → Docker sends SIGTERM, but app ignores it |

---

### 3️ The PID 1 Problem in Action

Try this demo 

```bash
cat <<'EOF' > zombie.sh
#!/bin/sh
while true; do
  sleep 0.1 &
done
EOF

chmod +x zombie.sh
docker run --rm alpine sh -c "./zombie.sh"
```

Now in another terminal:

```bash
docker exec -it <container_id> ps -ef
```

You’ll see dozens of `sleep` processes piling up — **zombies** that never die.
Your app (PID 1) isn’t calling `wait()` to reap them.

---

### 4️ Fixing It with `tini`

`tini` is a tiny init process that becomes PID 1, forwards signals properly, and reaps zombies.

#### Option A – Manual

```bash
docker run --init alpine ./zombie.sh
```

#### Option B – Dockerfile

```Dockerfile
FROM alpine
RUN apk add --no-cache tini
ENTRYPOINT ["/sbin/tini","--"]
CMD ["./app.sh"]
```

Now PID 1 = `tini`, PID 2 = your app 
`tini` handles signal forwarding and child cleanup.

---

### 5️ Signal Handling in Containers

By default Docker sends `SIGTERM` then `SIGKILL` to PID 1.
If your process doesn’t trap these, the container won’t exit cleanly.

Example Go code:

```go
signal.Notify(sigChan, syscall.SIGTERM)
<-sigChan
fmt.Println("Graceful shutdown…")
```

Then build and run inside a container — you’ll see the difference.

---

##  Labs — Step by Step

### Lab 11.1 — Observe PID 1

```bash
docker run -it alpine ps -ef
```

Output → `PID 1 sh`

---

### Lab 11.2 — Spawn Zombies

```bash
docker run -it alpine sh -c "while true; do sleep 0.1 & done"
```

Monitor → `docker exec ps -ef` → zombies accumulate.

---

### Lab 11.3 — Fix with Tini

```bash
docker run --init -it alpine sh -c "while true; do sleep 0.1 & done"
```

Now no zombies 

---

### Lab 11.4 — Signal Forwarding

```bash
docker run -d --name web nginx
docker stop web
```

Behind the scenes → Docker sends `SIGTERM → SIGKILL`
If your app ignores it, you’ll see forced termination.

---

##  Notes

* Always use `--init` or `tini` for long-running apps.
* Daemonized processes inside containers should handle signals and child reaping.
* `docker kill -s SIGUSR1` lets you test signal propagation.
* PID 1 inside a container has no parent → becomes its own reaper.
* Kubernetes uses `tini` by default via the pause container.

---

##  Summary

* Containers run your process as PID 1.
* PID 1 has special signal and reaping responsibilities.
* Lack of init leads to zombie build-up and shutdown issues.
* `tini` or `--init` solves both by becoming a mini init system.

---

###  Next Lesson → **Lesson 12: Healthchecks & Restart Policies**



Would you like me to generate a **dockAI-style solid diagram** for this lesson — showing the *PID 1 hierarchy*, comparing
**Without Init → With Tini**, including orphaned vs reaped child processes?
