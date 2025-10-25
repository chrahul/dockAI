
#  **Lesson 14: Logging, Monitoring & Events**

### *(Module 3 – Container Lifecycle & Runtime Behavior)*

---

##  **Context**

In production, your containers are talking all the time — through **stdout**, **stderr**, and **Docker events**.
If you can listen and interpret these signals well, you can **debug faster**, **prevent downtime**, and **build smarter observability pipelines**.

Docker provides native mechanisms for:

* Logging
* Monitoring resource usage
* Capturing lifecycle events (start, stop, OOM, kill, healthcheck)

This lesson will cover each in depth — both from the **developer’s** and **DevOps operator’s** perspective.

---

##  **Learning Goals**

 Understand how Docker logs are captured and stored
 Learn different logging drivers and when to use them
 Monitor containers using `docker stats`, `events`, and external collectors
 Integrate with ELK, Fluentd, or Loki stacks
 Simulate real-time event streams and debug patterns

---

##  **Concept**

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/5236dbf1-170a-49db-8790-88be1fb44f79" />


### 1️ The Logging Pipeline

When a container runs:

* Application writes to **stdout** / **stderr**
* Docker daemon captures this stream
* Logging driver decides where it’s stored

#### Default: `json-file`

Logs are written to:

```
/var/lib/docker/containers/<container_id>/<container_id>-json.log
```

Check log driver:

```bash
docker inspect <container> --format '{{.HostConfig.LogConfig.Type}}'
```

Output:

```
json-file
```

#### View Logs

```bash
docker logs <container_name>
docker logs -f <container_name>   # follow live logs
docker logs --since 1h <container>
```

---

### 2️ Log Rotation

Since logs are just files, they can grow huge.

Use `/etc/docker/daemon.json` to configure rotation:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

Restart daemon:

```bash
sudo systemctl restart docker
```

Now logs rotate after 10 MB, keeping 3 files.

---

### 3️ Other Logging Drivers

| Driver      | Description                 | Use Case                 |
| ----------- | --------------------------- | ------------------------ |
| `json-file` | Default local storage       | General dev use          |
| `syslog`    | Sends logs to system syslog | Legacy monitoring        |
| `fluentd`   | Streams logs to Fluentd     | Centralized aggregation  |
| `awslogs`   | Sends to CloudWatch         | AWS environments         |
| `gelf`      | Graylog Extended Log Format | Graylog/ELK              |
| `journald`  | Systemd journal             | Linux server integration |
| `local`     | Optimized local storage     | Better than json-file    |
| `none`      | Disable logging             | Silent containers        |

Example:

```bash
docker run -d --log-driver=fluentd nginx
```

---

### 4️ Live Monitoring – `docker stats`

```bash
docker stats
```

Output:

```
CONTAINER ID   NAME        CPU %   MEM USAGE / LIMIT   NET I/O   BLOCK I/O   PIDS
3c0dcd6e4a1b   webapp      2.15%   120MiB / 512MiB     1.2MB/1MB 3MB/2MB     10
```

You can combine this with `watch`:

```bash
watch -n 2 docker stats
```

---

### 5️ Docker Events — The Real-Time Feed

Every time a container changes state, Docker emits an **event**.

```bash
docker events
```

Output example:

```
2025-10-23T08:45:00.321Z container start 3c0dcd6e4a1b (image=nginx:latest, name=webapp)
2025-10-23T08:45:10.652Z container die 3c0dcd6e4a1b (exitCode=0)
2025-10-23T08:45:10.662Z network disconnect bridge (container=3c0dcd6e4a1b)
```

Filter specific event:

```bash
docker events --filter event=die
```

Or for a specific container:

```bash
docker events --filter container=webapp
```

---

### 6️ Integrating Logs into Centralized Systems

For large environments, logs are aggregated centrally for analysis and alerting.

Common pipelines:

* **Fluentd → Elasticsearch → Kibana (EFK)**
* **Promtail → Loki → Grafana**
* **Fluent Bit → CloudWatch / Datadog**

Example Docker Compose snippet:

```yaml
services:
  nginx:
    image: nginx
    logging:
      driver: "fluentd"
      options:
        fluentd-address: localhost:24224
        tag: docker.nginx
```

---

##  **Labs – Step by Step**

### Lab 14.1 — Inspect Log Files

```bash
docker run -d --name log-demo nginx
sudo tail -f /var/lib/docker/containers/$(docker ps -q)/$(docker ps -q)-json.log
```

Observe structured JSON logs in real time.

---

### Lab 14.2 — Use a Different Log Driver

```bash
docker run -d --name syslog-demo --log-driver=syslog nginx
```

Check system log:

```bash
sudo tail -f /var/log/syslog
```

---

### Lab 14.3 — Observe Events in Real Time

```bash
docker events --filter type=container
```

Now open another terminal:

```bash
docker stop log-demo
docker start log-demo
```

Watch events flow live.

---

### Lab 14.4 — Combine Stats + Events

Run:

```bash
docker stats &
docker events --filter container=log-demo &
```

Monitor both metrics and lifecycle changes simultaneously.

---

##  **Tips**

* Always use `--log-opt max-size` and `--log-opt max-file` in production.
* Integrate logs early in your CI/CD — don’t leave it as an afterthought.
* Use `docker system df` to monitor disk usage due to logs.
* Combine with **Prometheus node exporter** for full observability.

---

##  **Summary**

| Tool            | Purpose                         | Example                |
| --------------- | ------------------------------- | ---------------------- |
| `docker logs`   | View stdout/stderr output       | `docker logs -f nginx` |
| `docker stats`  | Monitor resource usage          | `docker stats`         |
| `docker events` | Track container lifecycle       | `docker events`        |
| Log Drivers     | Send logs to different backends | `--log-driver=fluentd` |

Logs tell you **what happened**,
Stats tell you **how it’s performing**,
Events tell you **when and why it changed**.

---

##  **Next Lesson → Lesson 15: Observability & Troubleshooting Deep Dive**

We’ll explore how to use `docker inspect`, `nsenter`, and low-level tools like `strace` and `tcpdump` to understand what’s really happening *inside* a container.



