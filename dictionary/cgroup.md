---
description: A Linux kernel feature that limits and measures the CPU, memory, I/O, and process count used by a group of processes.
---

A cgroup (control group) is a Linux kernel feature that limits and measures the resources used by a group of processes. Where a [Linux namespace](./Linux%20namespace.md) controls what a process can see, a cgroup controls how much it can use. [Container](./Container.md) runtimes create a cgroup for each one, and the [resource requests and limits](./Resource%20requests%20and%20limits.md) set in [Kubernetes](./Kubernetes.md) are enforced through them. Most current Linux distributions default to cgroup v2.

| Resource | Limit example                | What happens at the limit                     |
| -------- | ---------------------------- | --------------------------------------------- |
| CPU      | 0.5 CPU                      | Processes are paused until the next period    |
| Memory   | 512 MiB                      | Reclaim is attempted, then the OOM killer runs |
| PIDs     | 200 processes                | Creating another process or thread fails      |
| Block I/O | Bytes or operations per second | Reads and writes are slowed                 |

The two limits behave very differently. Exceeding a memory limit gets a process killed, which appears in Kubernetes as [OOMKilled](./OOMKilled.md). Exceeding a CPU limit only slows the process down, which makes it harder to notice.

CPU limits work as a quota per scheduling period, by default 100 ms. A limit of 0.5 CPU allows 50 ms of CPU time per period, counted across all threads. Four busy threads spend that in about 12.5 ms, then all of them are paused for the remaining 87.5 ms. The result is a service with latency spikes while its average CPU usage looks moderate. The metric to check is CPU throttling, reported in the cgroup's `cpu.stat` file.

Software inside the container may not know about these limits. Tools such as `free` read `/proc/meminfo`, which reports the host's memory. An application that sizes its heap or thread pool from host numbers can overrun its limit. Many language runtimes now read cgroup limits, but check yours.

_Usage:_

"P99 latency jumps every few seconds, but the CPU graph sits at 40%."

"Check throttling. The cgroup quota could be used up early in each period, and the threads then sit paused until the next one."
