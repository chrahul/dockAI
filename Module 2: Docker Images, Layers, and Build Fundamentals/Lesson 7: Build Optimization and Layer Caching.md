

#  Lesson 7: Build Optimization and Layer Caching

### *(Module 2 – Docker Images, Layers, and Build Fundamentals)*

---

##  Context

By now you know:

* Every Dockerfile instruction creates or modifies a **layer**.
* Docker caches layers based on **content hash + instruction + build context**.

This caching behavior is the **key to faster, smaller, reproducible builds**.
Lesson 7 teaches you how to design Dockerfiles that **reuse cache intelligently**, avoid unnecessary rebuilds, and reduce image size without breaking functionality.

We’ll also explore **multi-stage builds** — the professional way to ship minimal production images without keeping compilers or build tools inside.

---

##  Learning Goals

* Understand **how Docker decides cache hits or misses**.
* Optimize **layer order** and **instruction grouping** for efficiency.
* Implement **multi-stage builds** for size reduction.
* Use **build args** and **target stages** for flexible pipelines.
* Inspect cache behavior visually via `--progress=plain` and `docker history`.

---

##  Concept


<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/5b7d6f9d-7131-479b-8ffb-9042e71e26a2" />


### 1️ How Docker Cache Works

For each instruction, Docker computes a **cache key** from:

```
Instruction text + parent layer hash + files from build context
```

If nothing changes → cache reused.
If even a single bit changes → cache invalidated for that step and all below it.

You can see cache usage:

```bash
docker build --progress=plain .
```

 **Cache hit** = “Using cache”
 **Cache miss** = “Running command …”

---

### 2️ Why Layer Order Matters

Consider this order:

```Dockerfile
COPY . /app
RUN apk add --no-cache curl
```

Every code change triggers `COPY` → invalidates cache → re-runs `apk add`.

Better:

```Dockerfile
RUN apk add --no-cache curl
COPY . /app
```

System packages remain cached even when code changes.

---

### 3️ Combine and Minimize Layers

Each `RUN` adds a new layer. Combine related commands:

 Bad:

```Dockerfile
RUN apk add --no-cache git
RUN apk add --no-cache curl
```

 Good:

```Dockerfile
RUN apk add --no-cache git curl && rm -rf /var/cache/apk/*
```

---

### 4️ Multi-Stage Builds

Use temporary stages to compile or build artifacts, then copy only what’s needed.

Example:

```Dockerfile
# Stage 1: Build
FROM golang:1.22 AS builder
WORKDIR /src
COPY . .
RUN go build -o app .

# Stage 2: Runtime
FROM alpine:3.20
COPY --from=builder /src/app /usr/local/bin/app
CMD ["app"]
```

Advantages:
 Final image is tiny
 No compiler, no build tools
 Easier CI/CD caching per stage

Build specific stage:

```bash
docker build --target builder -t myapp:build .
```

---

### 5️ ARG, ENV and Cache Behavior

`ARG` values are part of cache keys, but `ENV` is runtime only.
Changing an `ARG` invalidates cache for that step and below.

Example:

```Dockerfile
ARG VERSION=1.0
RUN echo $VERSION > /version.txt
```

Rebuild with:

```bash
docker build --build-arg VERSION=2.0 .
```

→ Cache miss (rebuilds from that step).

---

### 6️ .dockerignore and Context Optimization

We saw in Lesson 6 that a smaller context = faster cache reuse.
Always exclude logs, build outputs, and VCS folders.

---

##  Labs — Step by Step

### Lab 7.1 — Observe Cache Hits and Misses

**Dockerfile**

```Dockerfile
FROM alpine:3.20
RUN echo "installing packages..." && apk add --no-cache curl
COPY app.sh /app.sh
CMD ["sh", "/app.sh"]
```

**app.sh**

```bash
echo "Hello Cache World"
```

Build #1:

```bash
docker build -t cache-demo:v1 --progress=plain .
```

**Output**

```
 => [2/3] RUN echo "installing packages..." && apk add ...   5.2s
 => [3/3] COPY app.sh /app.sh                                0.1s
```

Now rebuild without changing anything:

```
 => [2/3] RUN echo "installing packages..." ...               CACHED
 => [3/3] COPY app.sh /app.sh                                CACHED
```

 Both cached.

Change `app.sh`:

```bash
echo "Cache busted" > app.sh
docker build -t cache-demo:v2 --progress=plain .
```

Now only COPY (and subsequent steps) re-run.

---

### Lab 7.2 — Reorder for Smarter Caching

**Bad order**

```Dockerfile
FROM alpine:3.20
COPY . /app
RUN apk add --no-cache curl
```

Every code change invalidates `COPY` → re-runs package install.

**Optimized order**

```Dockerfile
FROM alpine:3.20
RUN apk add --no-cache curl
COPY . /app
```

Build once, then edit app file → package layer remains cached.

---

### Lab 7.3 — Multi-Stage Build Size Comparison

**Dockerfile**

```Dockerfile
# Builder
FROM golang:1.22 AS builder
WORKDIR /src
COPY . .
RUN go build -o app .

# Runtime
FROM alpine:3.20
COPY --from=builder /src/app /usr/local/bin/app
CMD ["app"]
```

Build:

```bash
docker build -t multistage:1.0 .
docker images multistage:1.0 golang:1.22 alpine:3.20
```

**Output**

```
REPOSITORY   TAG   SIZE
multistage   1.0   14MB
golang       1.22  1.18GB
alpine       3.20  7MB
```

 Over 99 % reduction in final size!

---

### Lab 7.4 — Using ARG and Target Builds

**Dockerfile**

```Dockerfile
FROM alpine:3.20 AS base
ARG VERSION=dev
RUN echo "Building version $VERSION"

FROM base AS prod
ARG VERSION=prod
RUN echo "Building version $VERSION"
```

Build different stages:

```bash
docker build --target base -t app:dev .
docker build --target prod -t app:prod .
```

Inspect logs → notice different `VERSION` per target.

---

### Lab 7.5 — Clean Intermediate Layers

Remove dangling images and unused layers after experiments:

```bash
docker system prune -f
docker builder prune -f
```

---

##   Notes

* **Layer reuse** = the biggest speed gain in Docker CI/CD.
* Place **OS/package installs early**, **app code late**.
* **Multi-stage builds** are your best friend for size optimization.
* Avoid using `latest` tag — it breaks cache predictability.
* Pin base images and package versions.
* Run `docker system df` regularly to monitor cache storage.

---

##  Summary

* Docker caching is deterministic: same input → same layer → reused.
* Reordering instructions can save minutes on every build.
* Multi-stage builds produce smaller, secure runtime images.
* `ARG`, `.dockerignore`, and build targets offer flexible, optimized pipelines.

---

###  Next Lesson → **Lesson 8: Modern Image Building with BuildKit and Buildx**

We’ll explore **parallel builds**, **multi-platform images**, **build secrets**, and **cache export/import** — the next-gen build system powering modern Docker pipelines.

---

