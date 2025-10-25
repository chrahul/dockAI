
#  Lesson 13: Resource Limits & Isolation

### *(Module 3 – Container Lifecycle & Runtime Behavior)*

---

##  Context

Containers aren’t virtual machines — they share the host kernel.
So how does Docker make sure one noisy container doesn’t starve others of CPU, RAM, or I/O?

The answer: **control groups (cgroups)** and **namespaces**.
They give containers the illusion of independence while keeping the system safe and predictable.

In this lesson, we’ll explore how Docker applies these limits, how to monitor resource usage, and how to diagnose throttling or out-of-memory kills.

---

##  Learning Goals

 Understand how Docker isolates containers using **cgroups v2**
 Apply and observe **CPU**, **memory**, and **I/O** limits
 Learn what happens during throttling or OOM kills
 Use commands like `docker stats`, `docker inspect`, and `cat /sys/fs/cgroup` for introspection

---

##  Concept

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/776ca821-a92e-45cc-837d-8a1523f4c3a5" />


### 1️ What Are cgroups?

Control Groups (cgroups) are a Linux kernel feature that **limit and account** for system resources per process.
Docker leverages these to assign quotas when containers start.

Each container gets its own directory under `/sys/fs/cgroup`, which includes sub-controllers:

* `cpu.max` — controls CPU shares and quotas
* `memory.max` — sets memory hard limits
* `pids.max` — limits number of processes
* `io.max` — restricts I/O throughput

---

### 2️ Namespaces Recap

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/317aefe4-510d-43ac-80f9-ecb3ac3d1dee" />


Namespaces isolate **what a process can see**.
cgroups isolate **what a process can use**.

| Namespace | Isolates                    |
| --------- | --------------------------- |
| PID       | Process IDs                 |
| NET       | Network interfaces          |
| MNT       | Filesystems                 |
| UTS       | Hostname/domain             |
| IPC       | Inter-process communication |
| USER      | UID/GID mappings            |

Together, namespaces + cgroups = lightweight isolation instead of full virtualization.

---

### 3️ Resource Limits in Action

#### CPU Limits

```bash
docker run -d --name cpu-test --cpus="1.5" ubuntu:22.04 sh -c "while true; do :; done"
```

This restricts the container to 1.5 CPU cores.

Inspect cgroup file:

```bash
cat /sys/fs/cgroup/<container_id>/cpu.max
```

**Output example:**

```
150000 100000
```

Meaning: 150ms CPU time allowed per 100ms period (1.5 CPUs).

---

#### Memory Limits

```bash
docker run -d --name mem-test --memory="256m" alpine sh -c "tail /dev/zero"
```

Monitor:

```bash
docker stats mem-test
```

When memory usage exceeds 256MB → **OOMKill** event occurs.

Inspect:

```bash
docker inspect --format='{{.State.OOMKilled}}' mem-test
```

Output → `true`

---

#### PIDs Limit

```bash
docker run -d --pids-limit=100 nginx
```

The container can spawn at most 100 processes.

---

### 4️ Observing and Monitoring

#### docker stats

Real-time metrics for all containers:

```bash
docker stats
```

Columns: CPU %, MEM USAGE/LIMIT, NET I/O, BLOCK I/O, PIDs

#### docker inspect

Detailed runtime info:

```bash
docker inspect <container> --format '{{json .HostConfig.Memory}}'
```

#### System cgroup view

```bash
cat /sys/fs/cgroup/system.slice/docker-<id>.scope/memory.current
```

---

### 5️ I/O Throttling

Example:

```bash
docker run -d --device-write-bps /dev/sda:10mb ubuntu dd if=/dev/zero of=/file bs=1M count=1000
```

This limits write throughput to 10MB/s.
Inspect with:

```bash
cat /sys/fs/cgroup/<id>/io.max
```

---

### 6️ cgroups v1 vs v2

Modern Docker (post-2023) uses **unified cgroups v2**:

* Single hierarchy for all controllers
* Better delegation and nesting
* Used by modern distros (Ubuntu 22+, RHEL 9, etc.)

Check version:

```bash
stat -fc %T /sys/fs/cgroup
```

Output:

```
cgroup2fs
```

---

##  Labs — Step by Step

### Lab 13.1 — CPU Limit Test

```bash
docker run -d --cpus="0.5" --name cpu-demo alpine sh -c "yes > /dev/null"
docker stats cpu-demo
```

Observe → CPU usage capped near 50% of one core.

---

### Lab 13.2 — Memory Limit and OOMKill

```bash
docker run -d --memory=200m alpine sh -c "python3 -c 'a=[0]*10**8'"
docker ps -a
docker inspect --format='{{.State.OOMKilled}}' <container>
```

Output → `true`
Container killed by kernel due to memory breach.

---

### Lab 13.3 — PID Limit Enforcement

```bash
docker run -d --pids-limit=10 alpine sh -c "while true; do sleep 0.1 & done"
```

After 10 forks → further attempts fail with `fork: Resource temporarily unavailable`.

---

### Lab 13.4 — Observe System Files

```bash
docker exec cpu-demo cat /sys/fs/cgroup/cpu.max
docker exec mem-test cat /sys/fs/cgroup/memory.max
```

---

##  Notes

* Always apply CPU/memory limits in production; unlimited containers can crash the host.
* Use `--cpus` instead of `--cpu-shares` for deterministic limits.
* `docker stats` is your real-time cgroup dashboard.
* If you see throttling, review `cpu.stat` or `memory.events`.
* Kubernetes converts these directly into Pod resource requests and limits.

---

##  Summary

* **Namespaces isolate visibility**, **cgroups control usage**.
* Docker applies resource limits transparently during `docker run`.
* You can monitor and debug them via `docker stats` and `/sys/fs/cgroup`.
* Proper limits = predictable, stable multi-container systems.

---

###  Next Lesson → **Lesson 14: Logging, Monitoring & Events**

We’ll explore how Docker collects logs, manages log drivers, and provides observability through `docker logs`, `events`, and external integrations.


