---
description: A process isolated by Linux kernel features, with its own filesystem and limits, sharing the host's kernel.
---

A container is a running process, or group of processes, that the Linux kernel isolates from the rest of the machine. It gets its own filesystem from a [container image](./Container%20image.md), its own view of the system through [Linux namespaces](./Linux%20namespace.md), and resource limits through [cgroups](./cgroup.md). It shares the host's kernel, which is the main difference from a virtual machine.

| Aspect    | Container                     | Virtual machine                       |
| --------- | ----------------------------- | ------------------------------------- |
| Kernel    | Shares the host's kernel      | Runs its own kernel                   |
| Boundary  | Kernel isolation features     | Hypervisor (software that runs VMs)   |
| Startup   | Starts a process              | Boots an operating system             |
| Footprint | Application and its libraries | Full operating system                 |

Sharing a kernel has practical effects. A Linux container needs a Linux kernel, so Docker Desktop on macOS and Windows runs one inside a lightweight virtual machine. A kernel vulnerability can affect every container on the host, so the isolation is weaker than a VM's. CPU architecture also still matters: an image built only for ARM fails on an x86 host with `exec format error` unless emulation is available.

A container lives only as long as its main process. When that process exits, the container stops, even if other processes were running inside it. If a script starts a background daemon and then finishes, the container exits immediately.

Containers are disposable, and their writable filesystem is deleted along with them. A common early mistake is writing uploads or a database file to the container's local disk, then finding the data gone after the next deploy replaced it. Data that must survive belongs in a mounted volume or an external service. Treating containers as replaceable is what makes [immutable infrastructure](./Immutable%20infrastructure.md) and [Kubernetes](./Kubernetes.md) scheduling workable.

_Usage:_

"All the uploaded avatars disappeared after this morning's deploy."

"They were saved inside the container, so they went away when it was replaced. Put them in object storage or a mounted volume."
