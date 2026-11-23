---
description: A Kubernetes object holding non-confidential settings that Pods read as environment variables or mounted files.
---

A ConfigMap is a [Kubernetes](./Kubernetes.md) object that stores non-confidential configuration as key-value pairs, separately from the [container image](./Container%20image.md). One image can then run in staging and production with different settings, which supports promoting a single [build artifact](./Build%20artifact.md). A ConfigMap is limited to 1 MiB and must be in the same namespace as the [Pods](./Pod.md) using it. Credentials belong in a [Kubernetes Secret](./Kubernetes%20Secret.md) or external [secrets management](./Secrets%20management.md).

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-config
data:
  LOG_LEVEL: info
  FEATURE_URL: https://flags.internal
```

The way a Pod consumes a ConfigMap decides what happens when it changes:

| Consumed as              | After the ConfigMap is edited                            |
| ------------------------ | -------------------------------------------------------- |
| Environment variable     | No change until the [container](./Container.md) restarts |
| Mounted volume           | Files update after a delay, and the app must re-read them |
| Volume with `subPath`    | No change until the container restarts                   |

The familiar symptom is editing a value, seeing the new value in `kubectl get configmap`, and finding that the application still behaves the old way. Editing a ConfigMap does not restart anything and does not trigger a [Kubernetes Deployment](./Kubernetes%20Deployment.md) rollout. Run `kubectl rollout restart deployment/api`, or include a hash of the config as an annotation in the Pod template so that a config change produces a new rollout. [Helm](./Helm.md) charts commonly do the second.

A shared, mutable ConfigMap also weakens [rollback](./Rollback.md), because rolling back the Deployment does not restore old config values. Versioned names such as `api-config-v12`, referenced from the Pod template, tie configuration to the release.

A missing ConfigMap blocks startup. A missing environment reference shows `CreateContainerConfigError`, and a missing volume leaves the Pod in `ContainerCreating` with a mount error in its events.

_Usage:_

"I changed LOG_LEVEL to debug an hour ago and still see nothing."

"It's injected as an env var from the ConfigMap, so running containers keep the old value. Do a rollout restart."
