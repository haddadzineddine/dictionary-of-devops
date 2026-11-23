---
description: A Kubernetes controller that keeps a set number of matching Pods running, usually created and managed by a Deployment.
---

A ReplicaSet is a [Kubernetes](./Kubernetes.md) object that keeps a specified number of matching [Pods](./Pod.md) running. If there are too few it creates more from its Pod template, and if there are too many it deletes the excess. It finds its Pods using a label selector, which is a query over key-value tags attached to objects.

You rarely create one by hand. A [Deployment](./Kubernetes%20Deployment.md) creates and owns ReplicaSets, one per version of the Pod template. That is why Pod names have two generated parts: in `api-7d9f6c5b4-x2kqp`, `7d9f6c5b4` identifies the ReplicaSet and `x2kqp` identifies the Pod.

| Object     | Responsible for                              |
| ---------- | -------------------------------------------- |
| Deployment | Versions, rollout strategy, and history      |
| ReplicaSet | The replica count for one version            |
| Pod        | Running the [containers](./Container.md)     |

During a [rolling deployment](./Rolling%20deployment.md), the Deployment scales a new ReplicaSet up and the old one down. Old ReplicaSets stay at zero replicas so that a [rollback](./Rollback.md) can scale one back up. `revisionHistoryLimit`, 10 by default, sets how many are kept. This is why `kubectl get rs` shows a list of entries with zero Pods, and they are normally safe to leave.

Three behaviours cause confusion. Manually scaling a ReplicaSet owned by a Deployment does not last, because the Deployment sets it back to the declared count. Editing the template of a standalone ReplicaSet does not update running Pods, because the template is only used when creating new ones, so the old [image](./Container%20image.md) keeps running until those Pods are replaced. A ReplicaSet can also adopt existing Pods that have no owner and match its selector, so overlapping labels can cause it to take over or delete Pods you created separately.

To change the replica count or image, edit the Deployment.

_Usage:_

"I scaled the ReplicaSet to five, and it went straight back to three."

"The Deployment owns it and still says three. Change the replica count on the Deployment."
