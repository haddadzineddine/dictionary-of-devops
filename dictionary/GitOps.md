---
description: Operating systems from declarative config in Git, with an agent that continuously pulls it and reconciles the live state.
---

GitOps is an operating model in which a Git repository holds the declared state of a system, and an automated agent continuously makes the live system match it. To change production, you merge a commit. Git supplies the review, audit history, and a way to revert, while the agent does the applying. Argo CD and Flux are common agents for [Kubernetes](./Kubernetes.md).

The model differs from a traditional deployment [pipeline](./Pipeline.md) in the direction of access:

| Aspect          | Push-based [CI](./Continuous%20integration.md)/CD                     | Pull-based GitOps                       |
| --------------- | ------------------------------------ | --------------------------------------- |
| Who applies     | The CI job                           | An agent inside the cluster             |
| Credentials     | CI holds cluster access              | Cluster holds read access to Git        |
| When it runs    | Once, on a pipeline trigger          | Continuously                            |
| Manual changes  | Remain until the next deploy         | Detected, and optionally reverted       |

A typical flow has CI build and publish a [container image](./Container%20image.md), then update the image tag in a separate configuration repository. The agent sees the commit and rolls out the change. A [rollback](./Rollback.md) is usually a `git revert` of that commit.

Continuous reconciliation can revert manual changes. During an incident, someone runs `kubectl scale` or `kubectl edit`, sees the change take effect, and a minute later it is gone. With self-healing enabled, the agent treats the manual edit as [configuration drift](./Configuration%20drift.md) and restores the Git version. To make an emergency change stick, commit it, or explicitly suspend syncing for that application and resume it afterwards.

Plain secrets cannot be stored in Git, so teams commit encrypted values or references to an external store through [secrets management](./Secrets%20management.md). Fields changed by controllers, such as replica counts under a [Horizontal Pod Autoscaler](./Horizontal%20Pod%20Autoscaler.md), must be excluded or they will fight the agent.

_Usage:_

"I scaled the API to ten replicas and it dropped back to four by itself."

"Argo reverted it. We use GitOps, so the repo wins. Open a PR changing the replica count."
