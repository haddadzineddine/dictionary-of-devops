---
description: A Kubernetes object that runs a set of identical Pods and manages rolling updates and rollbacks through ReplicaSets.
---

A [Kubernetes](./Kubernetes.md) Deployment is an object that declares how a stateless application should run: which [container image](./Container%20image.md), how many replicas, and how to update them. It does not manage [Pods](./Pod.md) directly. It creates a [ReplicaSet](./ReplicaSet.md) for each version of the Pod template, and shifts replicas between them during an update.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 3
  selector:
    matchLabels: { app: api }
  template:
    metadata:
      labels: { app: api }
    spec:
      containers:
        - name: api
          image: ghcr.io/acme/api:1.4.2
```

Changing anything in `template`, such as the image tag, triggers a [rolling deployment](./Rolling%20deployment.md):

| Step | What happens                                                        |
| ---- | ------------------------------------------------------------------- |
| 1    | A new ReplicaSet is created for the new template                    |
| 2    | It scales up, within the `maxSurge` allowance                       |
| 3    | Old Pods are removed as new ones pass their [readiness probe](./Readiness%20probe.md), within `maxUnavailable` |
| 4    | The old ReplicaSet stays at zero replicas, available for [rollback](./Rollback.md) |

Both `maxSurge` and `maxUnavailable` default to 25%. Changing only `replicas` scales the current ReplicaSet and does not start a rollout. Editing a [ConfigMap](./ConfigMap.md) does not start one either, which is why a config change can appear to do nothing.

A failed rollout often looks like a hang. If new Pods never become ready, the update stops partway and old Pods keep serving, which protects users. After `progressDeadlineSeconds`, 600 by default, the Deployment reports `ProgressDeadlineExceeded`. It does not roll itself back. Someone must fix the image or run `kubectl rollout undo`. [Pipelines](./Pipeline.md) should run `kubectl rollout status` so that a stuck rollout fails the job, because `kubectl apply` returns success as soon as the change is accepted.

Use a [StatefulSet](./StatefulSet.md) instead when replicas need stable identities or individual storage.

_Usage:_

"[CI](./Continuous%20integration.md) says the deploy passed, but we're still serving the old version."

"Apply only submits the change. The Kubernetes Deployment is stuck because the new pods are failing readiness. Check `kubectl rollout status`."
