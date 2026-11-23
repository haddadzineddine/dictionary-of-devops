---
description: A Kubernetes workload that runs one copy of a Pod on every eligible node, used for node-level agents.
---

A DaemonSet is a [Kubernetes](./Kubernetes.md) workload object that runs one copy of a [Pod](./Pod.md) on every eligible node. When a node joins the cluster it receives the Pod automatically, and when a node is removed its copy goes with it. There is no replica count to set, because the number of nodes determines the number of Pods.

DaemonSets suit software tied to the machine instead of to an application:

| Use                | Example job                                        |
| ------------------ | -------------------------------------------------- |
| Log collection     | Reads [container](./Container.md) log files from the node's disk     |
| Metrics            | Reports node CPU, memory, and disk                 |
| Networking         | Runs the cluster network plugin or kube-proxy      |
| Storage            | Runs a node driver that mounts volumes             |
| Security           | Watches system calls or file changes on the host   |

Such Pods often use `hostPath` volumes, which mount a directory from the node, or host networking, so a compromised agent can affect the whole node. Grant only the access it needs.

The usual complaint is an agent missing from some nodes, which leaves gaps in logs or metrics from those machines. The cause is often a taint, a marker on a node that repels Pods unless they declare a matching toleration. Dedicated GPU nodes and [control plane](./Control%20plane.md) nodes are commonly tainted. Compare `DESIRED` with the node count in `kubectl get daemonset`, then add the required toleration. A `nodeSelector` can also limit a DaemonSet deliberately to a subset of nodes.

A DaemonSet's cost multiplies by node count, because its [resource requests](./Resource%20requests%20and%20limits.md) are reserved on every machine.

Updates use a rolling strategy by default, replacing one node's Pod at a time (`maxUnavailable: 1`). A bad version of a networking or storage agent can degrade every node it reaches, so watch the first few and run `kubectl rollout undo` at the first failure.

_Usage:_

"We have no logs from the GPU nodes."

"Those nodes are tainted, so the logging DaemonSet never scheduled there. Add a toleration for the GPU taint."
