---
description: The smallest deployable unit in Kubernetes, made of one or more containers that share a network identity and can share storage.
---

A Pod is the smallest unit [Kubernetes](./Kubernetes.md) can deploy: one or more [containers](./Container.md) scheduled together on the same node. Containers in a Pod share a network namespace, so they have one IP address and reach each other on `localhost`, and they can mount the same volumes. Most Pods hold one application container, sometimes with a sidecar that ships logs or an init container that runs setup before the app starts.

Pods are disposable. A Pod is bound to one node and is not moved. If the node fails, or the Pod is deleted, a controller such as a [Deployment](./Kubernetes%20Deployment.md) creates a replacement with a new name and a new IP address. Code that stores a Pod IP in configuration breaks the next time that happens, so clients should connect through a [Service](./Kubernetes%20Service.md).

A restart is different from a replacement. When a container crashes, the kubelet (the agent on each node) restarts that container inside the same Pod, which keeps its name and IP. Files in the container's writable layer are lost, while an `emptyDir` volume survives until the Pod is removed.

| Phase     | Meaning                                                        |
| --------- | -------------------------------------------------------------- |
| Pending   | Accepted, but not all containers have started yet              |
| Running   | Bound to a node, with at least one container running or starting |
| Succeeded | All containers exited successfully and will not restart        |
| Failed    | All containers stopped, and at least one failed                |
| Unknown   | The Pod's state cannot be obtained, often a node problem       |

A Pod stuck in Pending usually has no node that fits its [resource requests](./Resource%20requests%20and%20limits.md), or it is waiting on a volume. The Events section of `kubectl describe pod` gives the reason. Statuses such as [CrashLoopBackOff](./CrashLoopBackOff.md) appear in `kubectl get pods` but are not phases.

_Usage:_

"The payment service had the worker's IP hardcoded, and it broke after the node drain."

"Pod IPs change whenever a pod gets replaced. Point it at the Service name instead."
