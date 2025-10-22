

#  Lesson 5: Docker Images, Layers, and the Union File System (AUFS & overlay2)

### *(Module 2 – Docker Images, Layers, and Build Fundamentals)*

---

##  Context

Module 1 explained **how Docker runs containers** (CLI → Daemon → containerd → runc → Kernel).
From this lesson onward, we open the box that containers are made from: **images**.

* What is a Docker image, *really*?
* Where do “layers” come from and how are they stored?
* Why do `RUN`, `COPY`, and `ADD` each create new layers?
* How does **Copy-on-Write (CoW)** make builds fast and images small?
* What’s inside `/var/lib/docker/overlay2` and how do *upper/lower/merged* dirs work?

By the end, you’ll be able to **read** Docker’s on-disk structures, **predict** cache hits/misses, and **debug** layer issues like a pro.

---

##  Learning Goals

* Understand **image anatomy**: manifest, config, layers (diff IDs vs chain IDs).
* Compare **AUFS** vs **overlay2** and why overlay2 is the modern default.
* Master **Union FS** concepts: *upper*, *lower*, *merged*, *work*, **whiteouts**, **opaque dirs**.
* Map **Dockerfile instructions → layers** and predict caching behavior.
* Navigate `/var/lib/docker/overlay2` safely and interpret what you see.
* Observe **Copy-on-Write** in action and verify with tools (`docker diff`, `mount`, `lsns`).

---

##  Concept

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/c7148b29-e4ee-42e6-a748-559576044681" />


### 1) What is a Docker Image?

A **content-addressable**, **read-only** stack of filesystem **layers**, plus metadata (config).

* **Manifest**: list of layer digests + config digest.
* **Config JSON**: OS/arch, env, entrypoint, history of commands.
* **Layers**: tar archives of filesystem *diffs* (not full filesystems).

Images are immutable. When you `docker run`, Docker creates a **thin writable layer** on top → that becomes the **container**.

```
[ Writable Container Layer ]   ← created at `docker run`
[ Image Layer N ]
[ Image Layer N-1 ]
...
[ Base Layer ]
```

---

### 2) Union File Systems: AUFS vs overlay2

Docker historically used **AUFS** (Ubuntu days). Modern Linux uses **overlay2**.

* **AUFS**: multi-branch union FS; not in mainline kernel; older distros.
* **overlay2**: built-in Linux OverlayFS; simpler, faster, maintained; supports *multiple lowerdirs*.
* **Takeaway**: On Linux, prefer **overlay2** (default on most distros and Docker CE).

Check yours:

```bash
docker info | grep -i 'Storage Driver'
# Storage Driver: overlay2
```

---

### 3) overlay2 anatomy (per container or mount)

OverlayFS merges **lower** (read-only) and **upper** (writable) trees into a **merged** view.

* **lowerdir**: read-only stack of image layers (may be dozens), de-duplicated by digest
* **upperdir**: the top writeable changes (container writes)
* **workdir**: required by kernel for CoW housekeeping
* **merged**: the unified mount path the process sees

Whiteouts and special markers:

* **Whiteout file**: `.wh.<filename>` in upperdir → hides a file present in lowerdir.
* **Opaque directory**: `trusted.overlay.opaque="y"` xattr on a dir in upperdir hides all lower entries.

---

### 4) Layers, Diff IDs, and Chain IDs

* **Diff ID**: SHA256 of the **uncompressed** layer tar (content).
* **Layer digest**: SHA256 of the **compressed** layer tar (registry reference).
* **Chain ID**: cumulative hash of a diff + all parents → uniquely identifies layer stack and enables de-dup.

Docker uses **content-addressable storage**: if two images share a layer digest, the tar exists **once** on disk.

---

### 5) When do Dockerfile instructions create layers?

* **Create a new layer**: `FROM` (base), each `RUN`, `COPY`, `ADD`
* **Do not create new layers**: `ENV`, `WORKDIR`, `EXPOSE`, `USER`, `ENTRYPOINT`, `CMD`, `LABEL`, `ARG` (affects build cache keys)

Order matters for **cache** and **size**. Put the most stable steps earlier.

---

### 6) How pulls and builds work (high level)

* **Pull**: fetch manifest → pull referenced layer tars (by digest) → verify → unpack to overlay2 layer dirs.
* **Build**: compute cache key for each step → reuse previous layer if key + context match; else run step → create new diff → store as new content-addressed layer.

---

##  Labs — Step by Step (with sample outputs)

> **Pre-req**: Linux host with Docker, sudo access, `jq` installed.

### Lab 5.1 — Identify your storage driver & basics

```bash
docker info | egrep -i 'Server Version|Storage Driver|Backing Filesystem|Root Dir'
```

**Sample output**

```
Server Version: 27.0.3
Storage Driver: overlay2
 Backing Filesystem: xfs
 Root Dir: /var/lib/docker
```

---

### Lab 5.2 — Pull a base image and inspect metadata

```bash
docker pull alpine:3.20
docker image inspect alpine:3.20 | jq '.[0] | {Id, RepoTags, Architecture, Os, RootFS, Config: {Env, Cmd}}'
```

**Sample output (trimmed)**

```json
{
  "Id": "sha256:0a97eee804...",
  "RepoTags": ["alpine:3.20"],
  "Architecture": "amd64",
  "Os": "linux",
  "RootFS": {
    "Type": "layers",
    "Layers": [
      "sha256:8c5698c2b5d3c0e4...",
      "sha256:1b1cebdfe2a9f0e3..."
    ]
  },
  "Config": {
    "Env": ["PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"],
    "Cmd": ["/bin/sh"]
  }
}
```

Note **RootFS Layers**: these are the *compressed* digests pulled from registry.

---

### Lab 5.3 — Explore overlay2 on disk (read-only image layers)

> **Careful**: Read only. Don’t edit files inside `/var/lib/docker`.

```bash
sudo ls -1 /var/lib/docker/overlay2 | head
# shows many UUID-like dirs (layer IDs)

docker image inspect alpine:3.20 | jq -r '.[0].RootFS.Layers[]' | head -n1
LAYER_DIGEST="sha256:8c5698c2b5d3c0e4..."

# Map layer digest to folder
sudo find /var/lib/docker/overlay2 -maxdepth 1 -type d -name '*8c5698*'
# /var/lib/docker/overlay2/8c5698c2b5d3c0e4.../
sudo tree -L 1 /var/lib/docker/overlay2/8c5698c2b5d3c0e4.../
# outputs: diff/  link  lower  work  merged (depending on whether mounted)
```

**Key**: `diff/` contains the **unpacked layer** content.
`lower` file points to parents; `merged` appears when a container is running using this mount.

---

### Lab 5.4 — Create a multi-layer image and inspect history

Create a simple Dockerfile:

```Dockerfile
# Dockerfile.layers
FROM alpine:3.20
RUN echo "layer1" > /msg.txt
RUN apk add --no-cache curl
COPY hello.txt /opt/hello.txt
ENV APP_ENV=dev
CMD ["cat","/msg.txt"]
```

Create `hello.txt`:

```bash
echo "hello layers" > hello.txt
```

Build:

```bash
docker build -f Dockerfile.layers -t layers-demo:1.0 .
```

Check history:

```bash
docker history layers-demo:1.0
```

**Sample output**

```
IMAGE          CREATED BY                                  SIZE     COMMENT
<image_id>     /bin/sh -c #(nop)  CMD ["cat" "/msg.txt"]   0B
<image_id>     /bin/sh -c #(nop)  ENV APP_ENV=dev          0B
<image_id>     /bin/sh -c #(nop) COPY file:... in /opt/... 15B
<image_id>     /bin/sh -c apk add --no-cache curl          6.2MB
<image_id>     /bin/sh -c echo "layer1" > /msg.txt         6B
<base_id>      /bin/sh -c #(nop)  ADD file:... in /        7.1MB
```

Note how **RUN/COPY** created new layers; **ENV/CMD** did not.

---

### Lab 5.5 — Observe Copy-on-Write at runtime

Run a container and change a file:

```bash
docker run -d --name demo layers-demo:1.0 sleep 600
docker exec demo sh -c 'cat /msg.txt && echo "runtime-change" >> /msg.txt && cat /msg.txt'
```

**Sample output**

```
layer1
layer1
runtime-change
```

Check what changed relative to the image:

```bash
docker diff demo
```

**Sample output**

```
C /msg.txt
A /opt/hello.txt  # shown if different
```

Inspect the mount (path will vary):

```bash
CID=$(docker inspect -f '{{.Id}}' demo)
sudo grep -R "$CID" /proc/*/mountinfo 2>/dev/null | head -n1
# ... overlay /var/lib/docker/overlay2/<upperid>/merged ...

UP=$(sudo readlink -f /var/lib/docker/overlay2/*/diff | head -n1)   # example only
sudo mount | grep overlay | head
```

Look into `upperdir` for the CoW copy of `/msg.txt`:

```bash
# Find container mount paths
docker inspect demo | jq '.[0].GraphDriver.Data'
```

**Typical output (overlay2)**

```json
{
  "LowerDir": "/var/lib/docker/overlay2/<lower1>:/var/lib/docker/overlay2/<lower2>:...",
  "MergedDir": "/var/lib/docker/overlay2/<mergeid>/merged",
  "UpperDir": "/var/lib/docker/overlay2/<upperid>/diff",
  "WorkDir": "/var/lib/docker/overlay2/<upperid>/work"
}
```

Now verify the changed file exists in **UpperDir**:

```bash
sudo ls -l /var/lib/docker/overlay2/<upperid>/diff/
# Shows msg.txt (the CoW copy)
```

---

### Lab 5.6 — Whiteouts & opaque dirs (deletions)

Delete a base-layer file and see how overlay hides it:

```bash
docker exec demo sh -c 'rm -f /opt/hello.txt'
docker diff demo
```

**Sample output**

```
D /opt/hello.txt
```

In the `UpperDir`, a **whiteout** like `.wh.hello.txt` will appear to mask the lower file.

```bash
sudo ls -a /var/lib/docker/overlay2/<upperid>/diff/opt
# .wh.hello.txt
```

---

### Lab 5.7 — Cache behavior experiment

Modify only `hello.txt` and rebuild:

```bash
echo "hello layers v2" > hello.txt
docker build -f Dockerfile.layers -t layers-demo:1.1 .
docker history layers-demo:1.1
```

Observe that **only the COPY step and below** were re-run (cache miss), earlier steps reused cache (cache hit).
Stable, early steps = faster builds.

---

### Lab 5.8 — Image sizes & real disk usage

```bash
docker images | grep layers-demo
docker system df -v
```

* `docker images` reports **virtual size** (sum of unique layers).
* `docker system df -v` shows **shared layers** across images.

---

##  Pro Notes & Gotchas

* **Minimize layers**: combine related `RUN` commands (but keep readability).
* **Order matters**: put `apt/apk install` and OS prep early; app code copy later.
* **.dockerignore** is critical: avoid sending huge build contexts (e.g., `node_modules`, `.git`).
* **COPY vs ADD**: prefer `COPY`; `ADD` only when you need tar auto-extract or remote URLs (rare).
* **Determinism**: pin versions and avoid time-dependent commands to keep cache stable.
* **Security**: unneeded tools (curl, compilers) inflate attack surface — remove them in same layer (`&& rm -rf /var/cache/...`).

---

##  Summary

* A Docker image is **metadata + stacked read-only diffs**; a container adds a **writable CoW layer**.
* **overlay2** is the modern, kernel-native UnionFS used by Docker.
* Mastering **upper/lower/merged/whiteouts** lets you reason about size, performance, and correctness.
* Build caching is **layer-based**: Dockerfile order and context control speed and size.

---

###  Next Lesson → **Lesson 6: Dockerfile Instructions & Build Context**

