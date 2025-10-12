# Docker Deep Dive - Ultimate Edition 2025
From Zero to Internals to Enterprise Deployments

---

## Module 1: Foundations of Docker and Containerization

- The Problem Before Containers (Inconsistencies, heavy VMs, long deployments)
- Evolution: Bare Metal -> Virtual Machines -> Containers
- Docker vs Virtualization
- Why Docker Revolutionized DevOps
- Docker Architecture Overview
  - Docker CLI, Daemon, API
  - Images, Containers, Layers, Registries

**Lab 1**
Install Docker, verify setup, run hello-world, inspect basic commands (docker ps, docker images, docker inspect).

---

## Module 2: Docker Images, Layers and Build Fundamentals

- Images, Layers and the Union File System (AUFS, overlay2)
  - How layers stack and merge
  - Copy-on-Write (CoW) explained
  - Inspect /var/lib/docker/overlay2
- Dockerfile Instructions (Complete Reference)
  - FROM, COPY, ADD, RUN, CMD, ENTRYPOINT, ENV, ARG, LABEL, USER, STOPSIGNAL, HEALTHCHECK, ONBUILD
- Build Context and .dockerignore
- Optimizing Images and Layer Caching

**Lab 2**
Build a custom Dockerfile, modify files, observe CoW layers, use .dockerignore, compare image sizes.

---

## Module 3: Modern Image Building (BuildKit and Buildx)

- Why BuildKit: Parallel builds, caching, secrets handling
- Enabling BuildKit (DOCKER_BUILDKIT=1)
- Multi-platform builds using Buildx (linux/amd64, arm64)
- Exporting and importing caches (--cache-to, --cache-from)
- Build secrets (--secret, --ssh)

**Lab 3**
Create a multi-platform build with Buildx, use cache export/import, test secret mounts.

---

## Module 4: Container Lifecycle and Runtime Behavior

- Container states: create, start, stop, restart, remove
- PID 1 problem, zombie processes and tini
- Healthchecks and Restart Policies
- Resource Limits (CPU, Memory, IO, PIDs)
- Logging Drivers and Log Rotation
- docker stats, docker events, docker cp, docker exec

**Lab 4**
Run a CPU-intensive container, apply limits, monitor throttling and restarts.

---

## Module 5: Storage, Volumes and Persistence

- Types of Storage: Named Volumes, Bind Mounts, tmpfs
- Storage Drivers (overlay2)
- Inspecting Mounts in /var/lib/docker
- Volume Lifecycle and Backup Strategies
- Volume Migration Between Hosts

**Lab 5**
Run MySQL with volume, backup and restore data to a new container, compare bind mount vs named volume.

---

## Module 6: Docker Networking Fundamentals

- Default Network Drivers
  - Bridge
  - Host
  - None
- Creating and Managing Networks
  - docker network create, inspect, connect, disconnect
- Container Communication
  - Internal DNS and container name resolution
- Exposing and Publishing Ports
  - --expose vs -p

**Lab 6**
Create a custom bridge network, connect containers, inspect IPs, test ping and curl between them.

---

## Module 7: Networking Deep Dive and Life of a Packet

How packets travel inside Docker.

### Lesson 7.1 Inside the Bridge Network
- docker0 bridge
- veth pairs and container interfaces
- NAT and iptables flow
- brctl, ip addr, and ip link commands

### Lesson 7.2 Host, None, and Macvlan Networks
- Host network performance and trade-offs
- Macvlan L2 access (assigning container real IPs)
- None network for isolation

### Lesson 7.3 Overlay Networks (Swarm and Multi-Host)
- VXLAN encapsulation
- Cross-host communication setup
- When overlay is useful even locally

### Lesson 7.4 Life of a Packet in Docker
- Step-by-step journey: Host NIC -> veth pair -> docker0 -> NAT -> container -> return
- Packet tracing using tcpdump, ss, iptables -t nat -L
- Practical visualization of ingress/egress flow

### Lesson 7.5 Rootless Networking
- Rootless Docker user-mode network stack
- Limitations and workarounds

**Labs**
1. Inspect veth pairs, MAC, and bridge mappings
2. Run containers under each network type and measure latency
3. Trace a packet with tcpdump -i docker0
4. Create two VMs, connect via overlay, verify VXLAN headers

---

## Module 8: Docker Internals and OS-Level Magic

What happens when you run "docker run nginx".

### Lesson 8.1 The Linux Foundations
- Namespaces: PID, NET, MNT, IPC, UTS, USER
  - How each isolates container resources
  - lsns, cat /proc/<pid>/ns, nsenter
- Cgroups v2
  - CPU, Memory, IO quotas
  - Observe throttling with top and docker stats
- Union File System and Copy-on-Write
  - overlay2 vs AUFS
  - Inspect mount layers (mount | grep overlay)
- Security Layers
  - seccomp filters
  - AppArmor and SELinux
  - Linux Capabilities (--cap-add, --cap-drop)
- Rootless Docker
  - Run containers without root privileges
  - Namespaces in user mode

**Labs**
1. Use nsenter to access container namespaces
2. Create CPU/memory limits and test enforcement
3. Drop network capabilities and observe blocked actions
4. Inspect overlay2 diff directories for file layer changes

---

## Module 9: Docker Compose and Multi-Service Applications

- YAML structure and service orchestration
- Compose v2, profiles, depends_on, health conditions
- .env files, variable substitution
- Staging vs Production patterns

**Lab 9**
Deploy WordPress and MySQL stack using Compose; test restart and persistence.

---

## Module 10: Registries, Images and Promotion Flow

- Docker Hub limits, orgs, tokens
- Private Registries (Harbor, ECR, ACR, GCR)
- Pull-through caching and mirroring
- Immutable tag strategy
- Registry garbage collection and cleanup

**Lab 10**
Run private registry container, push/pull images, configure caching proxy.

---

## Module 11: Supply Chain Security and Image Signing

- SBOM (Software Bill of Materials): Generate with Trivy or Syft
- Image Scanning: Docker Scout integration
- Image Signing: Notation v2, Cosign, Sigstore
- Provenance and Attestations
- CI/CD Enforcement: Reject unsigned or unscanned images

**Lab 11**
Generate SBOM, Sign with Cosign, Verify, Enforce in CI.

---

## Module 12: Engine Internals and Security Hardening

- OCI image spec and runtime
- Containerd vs runc
- Daemonless Docker architecture
- seccomp and AppArmor profiles
- Capabilities deep dive
- Rootless operation patterns
- Sandbox isolation on multi-tenant hosts

**Lab 12**
Inspect running containerd processes, view OCI runtime JSON, modify seccomp profiles.

---

## Module 13: Developer Experience and Tooling

- VS Code Dev Containers
- Docker Desktop optimizations (WSL2, resource limits)
- GPU containers (NVIDIA toolkit)
- Multi-arch issues on Apple Silicon

**Lab 13**
Containerize your dev environment; run AI workload in GPU container.

---

## Module 14: Observability, Logging and Troubleshooting

- Log drivers (json-file, syslog, fluentd)
- Centralized logging setup
- Metrics with docker stats, Prometheus and Grafana
- Common failure modes and recovery
- Tools: docker inspect, events, nsenter

**Lab 14**
Build logging stack; simulate crash loop, debug using logs and metrics.

---

## Module 15: CI/CD with Docker

- Jenkins pipeline integration
- Docker in Docker builds
- Multi-arch image push
- SBOM and signing gates
- Canary or Blue-Green deployments
- Rollback automation

**Lab 15**
End-to-end pipeline: build -> test -> SBOM -> sign -> deploy -> monitor.

---

## Module 16: Orchestration and Scaling (Swarm vs Kubernetes)

- Docker Swarm architecture, services, stacks, configs
- Overlay networks in Swarm
- Kubernetes handoff - Pods and Deployments
- Swarm vs K8s comparison

**Lab 16**
Set up Swarm, deploy a multi-node stack, test failover and scaling.

---

## Module 17: Platform-Specific and Ops Runbook

- Linux vs Windows containers
- Apple Silicon build issues
- Volume backup and restore scripts
- Incident playbooks: rollback and disaster recovery

---

## Final Capstone Project

Enterprise-Grade Application Stack with Secure CI/CD and Observability

- Multi-container microservice app
- Multi-arch BuildKit pipeline
- SBOM and Cosign verification
- Centralized monitoring stack
- Automated rollback and runbook documentation

---

### Distinctive Features of This Version

- Full Life of a Packet and Namespace internals
- Visual and command-based Linux OS-level introspection
- Practical security labs (seccomp, cgroups, CoW inspection)
- Real enterprise workflows (SBOM, Cosign, Docker Scout, Buildx)
- Covers beginner to kernel-level internals plus CI/CD

