---
description: A system that runs containers across a cluster of machines and continually works to match the state you declared.
---

Kubernetes (often written K8s) is an open-source system that runs [containers](./Container.md) across a group of machines, called a cluster. You describe the state you want, such as "three copies of this [image](./Container%20image.md), reachable on this port", and Kubernetes starts, places, connects, and replaces workloads to keep that state true.

A cluster has two parts. The [control plane](./Control%20plane.md) stores the desired state and makes decisions. Worker machines, called nodes, run the workloads inside [Pods](./Pod.md).

Most work happens through a small set of object types:

| Need                          | Object                                                    |
| ----------------------------- | --------------------------------------------------------- |
| Run a stateless app           | [Deployment](./Kubernetes%20Deployment.md)                |
| Run an app with stable identity | [StatefulSet](./StatefulSet.md)                         |
| Run one copy per node         | [DaemonSet](./DaemonSet.md)                               |
| Give Pods a stable address    | [Service](./Kubernetes%20Service.md)                      |
| Route outside HTTP traffic    | [Ingress](./Ingress.md)                                   |
| Supply settings               | [ConfigMap](./ConfigMap.md)                               |
| Supply credentials            | [Secret](./Kubernetes%20Secret.md)                        |

The behaviour that defines Kubernetes is reconciliation. Controllers, which are background loops, compare the declared state with the actual state and act on any difference. For example, you delete a misbehaving Pod and a new one appears within seconds. You removed the instance but left the instruction that says it should exist. To stop it, scale down or delete the object that owns it.

Reconciliation also means that changes are asynchronous. A successful `kubectl apply` only records your intent in the API. It does not mean the app is running, so a deploy can "succeed" while every new Pod fails to start. Check the rollout status and Pod events afterwards.

Kubernetes adds real operational weight: networking, upgrades, access control, and many concepts to learn. A managed offering such as Amazon EKS, Google GKE, or Azure AKS runs the control plane for you, but your team still owns the workloads. A single small service often does not need it.

_Usage:_

"I keep deleting this broken pod and it keeps coming back."

"That's Kubernetes doing its job. The Deployment still asks for three replicas, so scale that down or fix the image."
