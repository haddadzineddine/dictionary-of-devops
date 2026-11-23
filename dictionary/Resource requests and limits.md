---
description: Per-container CPU and memory settings. Requests guide Pod placement, and limits cap what a container may use at runtime.
---

Resource requests and limits are per-[container](./Container.md) CPU and memory settings in [Kubernetes](./Kubernetes.md). A request is the amount the scheduler reserves when choosing a node. A limit is the ceiling enforced at runtime through [cgroups](./cgroup.md). CPU is measured in cores or millicores (`500m` is half a CPU), and memory in bytes (`256Mi`).

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    memory: 512Mi
```

| Aspect             | Request                                     | Limit                       |
| ------------------ | ------------------------------------------- | --------------------------- |
| Used by            | Scheduler, and the kernel for CPU shares    | Kernel at runtime           |
| Exceeding CPU      | Allowed when the node has spare CPU         | Throttled (paused)          |
| Exceeding memory   | Allowed, but raises eviction risk           | Container is [OOMKilled](./OOMKilled.md) |

Scheduling uses requests, not actual usage. A [Pod](./Pod.md) stays Pending with an `Insufficient cpu` or `Insufficient memory` event when no node has enough unreserved capacity, even if the machines are mostly idle. Requests set too low let the scheduler pack too many Pods on one node.

If only a limit is set, the request defaults to the same value. The settings also determine a Pod's quality of service class:

| Class      | Condition                                          |
| ---------- | -------------------------------------------------- |
| Guaranteed | Every container has CPU and memory requests equal to limits |
| Burstable  | At least one request or limit, but not Guaranteed  |
| BestEffort | No requests or limits at all                       |

When a node runs short of memory, the kubelet evicts Pods, generally starting with those using more than their request. BestEffort Pods have no request, so they are exposed first.

A common starting point is to set memory requests from observed usage, with a limit at or slightly above that value. CPU limits can throttle a service while the node has idle CPU, so some teams set only a CPU request. The [Horizontal Pod Autoscaler](./Horizontal%20Pod%20Autoscaler.md) calculates CPU utilisation against the request.

_Usage:_

"The pod's stuck in Pending, but the nodes are only at 20% CPU."

"Scheduling looks at requests, not usage. Those nodes are fully reserved, and this pod is asking for four cores. Lower its resource requests."
