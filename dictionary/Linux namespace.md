---
description: A Linux kernel feature that gives a process its own isolated view of a resource such as process IDs, network, or mounts.
---

A Linux namespace is a kernel feature that gives a group of processes their own isolated view of one system resource. A [container](./Container.md) is largely an ordinary process placed in a set of these namespaces, with [cgroups](./cgroup.md) limiting what it can consume.

| Namespace | Isolates                         | Effect inside a container                    |
| --------- | -------------------------------- | -------------------------------------------- |
| PID       | Process IDs                      | Main process appears as PID 1                |
| Network   | Interfaces, IPs, ports, routes   | Has its own `localhost`                      |
| Mount     | Filesystem mount points          | Sees the [image](./Container%20image.md) as its root filesystem |
| UTS       | Hostname                         | Has its own hostname                         |
| IPC       | Shared memory, message queues    | Cannot reach other containers' queues        |
| User      | User and group IDs               | Root inside can map to non-root outside      |
| Cgroup    | View of the cgroup hierarchy     | Sees its own cgroup as the root              |
| Time      | Boot and monotonic clocks        | Can have offset clocks                       |

The network namespace explains a frequent beginner failure. An app in one container connects to a database at `localhost:5432` and gets "connection refused", although the database is running. `localhost` refers to the container's own namespace, where nothing listens on that port. Connect using the other container's service name instead. Inside a [Kubernetes](./Kubernetes.md) [Pod](./Pod.md), the reverse applies: containers share one network namespace, so they do reach each other on `localhost`.

The PID namespace explains slow shutdowns. The kernel treats PID 1 specially: a signal with no installed handler is ignored instead of taking its default action. An app that never registers a `SIGTERM` handler therefore keeps running when asked to stop, and the runtime force-kills it after a timeout. Add a handler, or run a small init process such as `tini` as PID 1.

Namespaces hide resources but do not make a process secure by themselves. All containers on a host still share one kernel.

_Usage:_

"The API can't reach Redis on localhost, but Redis is definitely up."

"Different containers, different network namespaces. `localhost` is the API container itself. Use the `redis` hostname."
