---
description: A periodic Kubernetes health check that restarts a container when it fails repeatedly.
---

A liveness probe is a periodic health check, run by the kubelet (the agent on each node), that decides whether a [container](./Container.md) should be restarted. When the probe fails `failureThreshold` times in a row, 3 by default, the kubelet kills the container and the [Pod](./Pod.md)'s restart policy applies. The Pod stays on the same node and keeps its IP address.

```yaml
livenessProbe:
  httpGet:
    path: /livez
    port: 8080
  periodSeconds: 10
  timeoutSeconds: 2
  failureThreshold: 3
```

A probe can be an HTTP request (a status from 200 to 399 passes), a TCP connection, a gRPC health check, or a command that must exit with 0.

Liveness probes exist for failures that a restart can fix, such as a deadlock where the process is still running but no longer doing work. Without one, [Kubernetes](./Kubernetes.md) only restarts a container when its process exits.

A badly designed probe can cause an outage. A frequent mistake is having the liveness endpoint check the database. When the database slows down, every replica fails its probe and restarts together, removing capacity and adding a burst of reconnections. A restart cannot fix a dependency, so the liveness check should only cover the process itself. Dependency checks belong in a [readiness probe](./Readiness%20probe.md), which takes a Pod out of traffic without restarting it.

Timing causes the other common failure. An application that needs a minute to start gets killed before it finishes, restarts, and is killed again, which shows up as [CrashLoopBackOff](./CrashLoopBackOff.md). Add a startup probe, which holds off liveness and readiness checks until it first succeeds. The default `timeoutSeconds` is 1, so a busy but healthy process can miss probes under load and be restarted.

To confirm the cause, look for `Liveness probe failed` in the events from `kubectl describe pod`.

_Usage:_

"Every pod restarted at once when Postgres had that slow spell."

"Our liveness probe queries the database. Move that check to readiness, and have liveness confirm only that the process responds."
