
We’ve now completed:
**Module 1:** Foundations of Docker and Containerization
**Module 2:** Docker Images, Layers & Build Fundamentals

So yes — we now move to ** Module 3: Container Lifecycle and Runtime Behavior**

This is one of the most *fascinating* parts of Docker — it’s where we go **inside the running container** and understand what truly happens after `docker run`.

---

##  **Module 3 Overview — Container Lifecycle & Runtime Behavior**

> “If Module 2 was about *how Docker builds containers*,
> Module 3 is about *how they live, breathe, and die*.”

---

###  **What You’ll Learn**

| Lesson        | Theme                              | Focus                                                                     |
| ------------- | ---------------------------------- | ------------------------------------------------------------------------- |
| **Lesson 10** | Container Lifecycle                | From `docker run` to `docker stop` — the complete container state machine |
| **Lesson 11** | PID 1 Problem & Process Management | Why PID 1 behaves differently inside containers                           |
| **Lesson 12** | Healthchecks & Restart Policies    | Making containers self-healing                                            |
| **Lesson 13** | Resource Limits & Isolation        | CPU, memory, PIDs, blkio — powered by cgroups v2                          |
| **Lesson 14** | Logging, Monitoring & Events       | How Docker captures stdout/stderr, drivers, and event streams             |

---

###  **Core Concepts Covered**

* Container state transitions: *created → running → paused → stopped → removed*
* Lifecycle commands (`start`, `stop`, `restart`, `rm`)
* PID 1, zombie processes, and `tini`
* Restart policies (`always`, `unless-stopped`, `on-failure`)
* Healthchecks for self-healing workloads
* Resource control using **cgroups v2**
* Log drivers (`json-file`, `syslog`, `fluentd`, `awslogs`)
* Observability with `docker stats`, `events`, and `inspect`

---

###  **Labs Snapshot**

* Observe lifecycle transitions with `docker ps -a`
* Simulate zombie processes and fix using `tini`
* Configure restart policies and test container recovery
* Apply CPU and memory limits; trigger throttling
* Capture and rotate logs; explore logging drivers

---

###  **By the End of Module 3**

You’ll understand **how a container behaves at runtime**,
how Docker interacts with the Linux kernel (PID, namespaces, cgroups),
and how to monitor and control running workloads like a pro.

---

