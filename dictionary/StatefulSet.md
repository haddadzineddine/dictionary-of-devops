---
description: A Kubernetes workload giving each Pod a stable name, stable storage, and ordered rollout, for apps like databases.
---

A StatefulSet is a [Kubernetes](./Kubernetes.md) workload object for applications whose replicas are not interchangeable, such as databases, message brokers, and consensus systems. Each [Pod](./Pod.md) gets a stable ordinal name (`db-0`, `db-1`, `db-2`) and its own persistent volume. If `db-1` is replaced, the new Pod is also `db-1` and reattaches the same storage.

| Aspect       | [Deployment](./Kubernetes%20Deployment.md) | StatefulSet                   |
| ------------ | ------------------------------------------ | ----------------------------- |
| Pod names    | Random suffix                              | Stable ordinal                |
| Storage      | Usually shared or none                     | One volume claim per Pod      |
| Start order  | Parallel                                   | In order, 0 to N-1 by default |
| Update order | Surge and replace                          | Reverse order, N-1 to 0       |
| Network      | Shared [Service](./Kubernetes%20Service.md) address | Per-Pod DNS name via a headless Service |

A headless Service (one with no virtual IP) gives each replica an address such as `db-0.db.prod.svc.cluster.local`, so peers and clients can reach a specific member. Storage is declared in `volumeClaimTemplates`, which create one PersistentVolumeClaim (a request for durable storage) per Pod.

A StatefulSet provides identity and ordering only. Replication, leader election, and failover belong to the software itself or to a [Kubernetes Operator](./Kubernetes%20Operator.md).

Three behaviours often cause problems. Scaling down or deleting a StatefulSet leaves its volumes behind by default, and a later scale-up reattaches the old data. When a node becomes unreachable, its Pod is not recreated elsewhere, because two Pods with one identity could corrupt a shared volume. It stays stuck until the node returns or someone confirms the node is gone. With default ordering, a Pod that never becomes ready blocks the rollout, and it can stay blocked after the template is fixed until the broken Pod is deleted.

_Usage:_

"`db-2` has been Terminating for twenty minutes since that node died. Why isn't it rescheduled?"

"A StatefulSet won't start a second `db-2` until it knows the first is gone. Confirm the node is really dead before forcing it."
