---
description: The Kubernetes components that store cluster state, schedule workloads, and run the controllers that keep it as declared.
---

The control plane is the set of [Kubernetes](./Kubernetes.md) components that stores cluster state and decides what should run where. Worker nodes run the applications. The control plane tells them what to run and tracks the result.

| Component                | Role                                                                 |
| ------------------------ | -------------------------------------------------------------------- |
| API server               | Front door for all reads and writes, with authentication and validation |
| etcd                     | Consistent key-value store that holds all cluster state              |
| Scheduler                | Picks a node for each unscheduled [Pod](./Pod.md)                    |
| Controller manager       | Runs the loops that reconcile actual state with desired state        |
| Cloud controller manager | Talks to the cloud provider for load balancers, routes, and nodes    |

Each worker node runs a kubelet, an agent that watches the API server and starts the Pod's [containers](./Container.md) through a container runtime. Components communicate through the API server, and only the API server talks to etcd.

A typical sequence for creating a [Deployment](./Kubernetes%20Deployment.md): the API server validates and stores it in etcd, a controller creates a [ReplicaSet](./ReplicaSet.md) and Pods, the scheduler assigns each Pod to a node, and that node's kubelet starts it.

Control plane failure does not immediately stop applications. Containers on healthy nodes keep running and can keep serving traffic, while `kubectl` commands fail or time out. The damage is that nothing can change: no new deployments, no scaling, and no rescheduling of Pods from a node that dies.

etcd needs a majority of its members to accept writes, so production clusters run an odd number, usually three or five. Three members tolerate one failure, and five tolerate two. Losing etcd without a backup means losing the cluster's definition, so self-managed clusters need tested backups. Managed services such as EKS, GKE, and AKS operate the control plane for you.

_Usage:_

"kubectl is timing out. Is the site down?"

"No, the app's still serving. The control plane is having trouble, so running pods are fine, but hold off on deploys until it recovers."
