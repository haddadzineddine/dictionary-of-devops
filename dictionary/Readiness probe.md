---
description: A periodic Kubernetes check that removes a Pod from Service endpoints while it fails, without restarting it.
---

A readiness probe is a periodic check, run by the kubelet (the agent on each node), that decides whether a [Pod](./Pod.md) should receive traffic. While it fails, the Pod is marked not ready and removed from the endpoints of any [Service](./Kubernetes%20Service.md) that selects it. The [container](./Container.md) keeps running and is not restarted. When the probe passes again, the Pod returns to rotation.

[Kubernetes](./Kubernetes.md) has three probe types, which answer different questions:

| Probe                                 | Question                  | On failure                        | Runs                  |
| ------------------------------------- | ------------------------- | --------------------------------- | --------------------- |
| Startup                               | Has it finished starting? | Container restarted at threshold  | Until first success   |
| [Liveness](./Liveness%20probe.md)     | Is the process working?   | Container restarted               | After startup         |
| Readiness                             | Can it take traffic now?  | Removed from Service endpoints    | After startup, continuously |

Without a readiness probe, a Pod counts as ready once its containers start. Traffic then reaches an application that is still loading data or opening connections, and users see a burst of errors on every deploy.

Readiness also controls [rolling deployments](./Rolling%20deployment.md). A [Deployment](./Kubernetes%20Deployment.md) only continues replacing old Pods as new ones become ready, so a version that never passes stalls the rollout while the old version keeps serving. That protection disappears if the probe always returns success.

Shared dependencies need caution. If every replica's readiness depends on one cache and that cache goes down, all Pods leave the endpoints at once, and a partial degradation becomes a full outage. Consider serving reduced functionality instead.

Shutdown is a separate gap. When a Pod is deleted, endpoint removal and the `SIGTERM` signal happen at about the same time, so a few requests can still arrive after the process begins to exit. Keep serving briefly after `SIGTERM`, or add a short `preStop` delay.

_Usage:_

"We get a spike of 503s for about thirty seconds on every deploy."

"The new pods take traffic before the cache warms up. Add a readiness probe that passes only once it's loaded."
