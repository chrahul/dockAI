

#  Lesson 6: Dockerfile Instructions & Build Context

### *(Module 2 — Docker Images, Layers, and Build Fundamentals)*

---

##  Context

In Lesson 5, we learned **how Docker images are structured** internally — stacks of layers built through Copy-on-Write (CoW).
Now it’s time to understand **how those layers are created** — line-by-line, instruction-by-instruction, inside a **Dockerfile**.

Every Dockerfile is like a recipe:
each command contributes an ingredient, a step, or a flavor to your final image.
The art lies in **ordering, caching, and context** — knowing what belongs inside the build context and what shouldn’t even be sent to Docker at all.

In the real world, 90 % of Docker performance and image-size issues come from just three mistakes:

1. Misunderstanding **build context**
2. Misusing **layer-creating instructions**
3. Forgetting to use **.dockerignore**

This lesson fixes all three.

---

##  Learning Goals

* Understand every major Dockerfile instruction and its effect on **layers** and **cache**.
* Learn how **build context** and **.dockerignore** influence performance.
* Design efficient, production-grade Dockerfiles that build fast and remain portable.
* Explore the difference between `CMD` and `ENTRYPOINT`.
* Observe layer creation and caching in real builds.

---

##  Concept


<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/d640b8bd-95e6-44a7-a936-0a26c91c5482" />


### 1️ The Build Context

When you run:

```bash
docker build -t myapp .
```

the **dot (`.`)** is critical — it means *“send this directory and everything under it”* to the Docker daemon as the **build context**.

So if you’re inside a project that’s 1 GB with node_modules, logs, and .git — all of that is **tarred and uploaded** to the Docker daemon.

You can verify what gets sent:

```bash
DOCKER_BUILDKIT=1 docker build --progress=plain .
```

 **Problem:** slow builds, huge cache keys, and leaking credentials.
 **Solution:** use `.dockerignore`.

#### Example `.dockerignore`

```
.git
node_modules
*.log
*.tmp
Dockerfile
```

---

### 2️ Dockerfile Syntax — High-Level Structure

Each Dockerfile follows this top-to-bottom flow:

```
FROM        # Base image
LABEL       # Metadata
ENV         # Environment variables
RUN         # Commands that modify filesystem
COPY/ADD    # Bring files from context
WORKDIR     # Change default working directory
EXPOSE      # Document listening ports
ENTRYPOINT  # Fixed command
CMD         # Default arguments
```

Each instruction becomes either:

* A **layer-creating** instruction (`RUN`, `COPY`, `ADD`, `FROM`)
* A **metadata-only** instruction (`ENV`, `EXPOSE`, `CMD`, etc.)

---

### 3️ FROM — the Foundation

Every Dockerfile starts with:

```Dockerfile
FROM alpine:3.20
```

You can even have **multi-stage builds**:

```Dockerfile
FROM golang:1.22 AS builder
WORKDIR /app
COPY . .
RUN go build -o app .

FROM alpine:3.20
COPY --from=builder /app/app /usr/local/bin/app
CMD ["app"]
```

This drastically reduces final image size (we’ll deep dive in Lesson 7).

---

### 4️ RUN — executes shell commands in the image

Each `RUN` layer executes inside a shell of the base image.

Example:

```Dockerfile
RUN apk add --no-cache curl
```

 Creates a new **layer**
 Adds curl binaries permanently
 Always clean caches in the same command:

```Dockerfile
RUN apk add --no-cache curl && rm -rf /var/cache/apk/*
```

Multiple `RUN` lines = multiple layers.
You can chain them with `&&` to save space — but balance it with readability.

---

### 5️ COPY vs ADD

| Feature                         | COPY          | ADD           |
| ------------------------------- | ------------- | ------------- |
| Copy local files                | Yes             | Yes             |
| Extract local tar automatically | No             | yes             |
| Download from remote URL        | No             | Yes             |
| Simplicity                      |  Recommended |  Use rarely |

Best practice:

```Dockerfile
COPY . /app
```

NOT

```Dockerfile
ADD . /app
```

unless you *need* the tar extraction.

---

### 6️ CMD vs ENTRYPOINT

| Behavior                                    | CMD          | ENTRYPOINT       |
| ------------------------------------------- | ------------ | ---------------- |
| Can be overridden by `docker run` arguments | Yes            | No                |
| Can use in combination                      | Yes            | Yes               |
| Typical usage                               | default args | fixed executable |

Example:

```Dockerfile
ENTRYPOINT ["python3", "app.py"]
CMD ["--debug"]
```

Then you can override CMD:

```bash
docker run myapp --prod
```

Output:

```
python3 app.py --prod
```

But ENTRYPOINT remains fixed.

---

### 7️ WORKDIR, ENV, USER, EXPOSE

```Dockerfile
WORKDIR /app
ENV PATH="/app/bin:${PATH}"
USER nobody
EXPOSE 8080
```

These don’t create new layers of data, only metadata (affect future instructions and runtime environment).

---

### 8️ LABEL and ARG

```Dockerfile
LABEL maintainer="rahul.chaubey@dockai"
ARG BUILD_VERSION=1.0
ENV VERSION=$BUILD_VERSION
```

`ARG` is available **only at build time**, not runtime.

---

### 9️ HEALTHCHECK

```Dockerfile
HEALTHCHECK --interval=30s --timeout=5s CMD curl -f http://localhost:8080/health || exit 1
```

This adds metadata so Docker can report container health.

---

##  Labs — Step by Step

> **Pre-req:** Linux system with Docker installed; `nano` and `curl` available.

---

### Lab 6.1 — Basic Dockerfile

Create the following Dockerfile:

```Dockerfile
# Dockerfile
FROM alpine:3.20
RUN echo "Hello from Dockerfile Lesson 6" > /hello.txt
CMD ["cat", "/hello.txt"]
```

Build it:

```bash
docker build -t lesson6:v1 .
```

Run it:

```bash
docker run --rm lesson6:v1
```

**Output**

```
Hello from Dockerfile Lesson 6
```

Check layers:

```bash
docker history lesson6:v1
```

**Output**

```
IMAGE          CREATED BY                                 SIZE
<image_id>     /bin/sh -c #(nop)  CMD ["cat" "/hello.txt"]   0B
<image_id>     /bin/sh -c echo "Hello..." > /hello.txt       6B
<base_id>      /bin/sh -c #(nop) ADD file:... in /           7.1MB
```

---

### Lab 6.2 — Using COPY and .dockerignore

Create project structure:

```
myapp/
 ├─ Dockerfile
 ├─ app.sh
 ├─ secrets.env
 ├─ node_modules/
 └─ .dockerignore
```

**Dockerfile**

```Dockerfile
FROM alpine:3.20
COPY . /app
CMD ["ls", "/app"]
```

**.dockerignore**

```
node_modules
secrets.env
```

Build:

```bash
docker build -t lesson6:copy .
```

Run:

```bash
docker run --rm lesson6:copy
```

**Output**

```
Dockerfile
app.sh
.dockerignore
```

Notice `node_modules` and `secrets.env` were excluded.

---

### Lab 6.3 — Understanding Build Context Size

```bash
docker build -t bigbuild . --no-cache
```

Observe in output:

```
Sending build context to Docker daemon  2.3MB
```

Add `.dockerignore` and rebuild — notice context size drops to KBs.

---

### Lab 6.4 — CMD vs ENTRYPOINT demo

Create Dockerfile:

```Dockerfile
FROM alpine:3.20
ENTRYPOINT ["echo", "ENTRYPOINT fixed:"]
CMD ["default"]
```

Build and test:

```bash
docker build -t lesson6:entry .
docker run --rm lesson6:entry
docker run --rm lesson6:entry custom
```

**Output**

```
ENTRYPOINT fixed: default
ENTRYPOINT fixed: custom
```

---

### Lab 6.5 — Multi-Layer and Cache Behavior

Create `Dockerfile.cache`:

```Dockerfile
FROM alpine:3.20
RUN echo "layer 1" > /msg1.txt
RUN echo "layer 2" > /msg2.txt
```

Build:

```bash
docker build -f Dockerfile.cache -t cachetest:v1 .
```

Now modify only the second command:

```Dockerfile
RUN echo "layer 2 changed" > /msg2.txt
```

Rebuild:

```bash
docker build -f Dockerfile.cache -t cachetest:v2 .
```

**Output Snippet**

```
 => [1/3] FROM alpine:3.20                            0.0s
 => [2/3] RUN echo "layer 1" > /msg1.txt              0.0s  (cached)
 => [3/3] RUN echo "layer 2 changed" > /msg2.txt      0.2s
```

Only the last layer rebuilt — caching confirmed.

---

### Lab 6.6 — Combine Commands for Efficiency

Bad:

```Dockerfile
RUN apk add --no-cache git
RUN apk add --no-cache curl
```

 2 layers, bigger image.

Good:

```Dockerfile
RUN apk add --no-cache git curl
```

 1 layer, smaller image.

---

##   Notes

* Always keep **Dockerfile and .dockerignore together**.
* **Lower layers = base OS, stable**; put less-changing steps earlier for caching.
* Avoid copying the whole repo; use **targeted COPY**.
* Use **multi-stage builds** to keep final images minimal.
* Remember: every `RUN`, `COPY`, `ADD` = **new layer**.
* Prefer **explicit versions** in package installs for reproducibility.

---

##  Summary

* Docker builds images layer by layer from top to bottom.
* Build context determines what’s sent to Docker — clean it with `.dockerignore`.
* Some instructions create filesystem layers, others only metadata.
* Cache efficiency depends on order and determinism.
* ENTRYPOINT and CMD define how your container runs.

---

###  Next Lesson → **Lesson 7: Build Optimization and Layer Caching**

We’ll dive deeper into **multi-stage builds**, **cache reuse**, **layer re-ordering**, and **slimming images** for production pipelines.

---
