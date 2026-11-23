---
description: A release strategy that replaces instances in small batches, so the service stays available while two versions run together.
---

A rolling deployment is a release strategy that replaces running instances of an application with a new version in small batches. Capacity stays mostly available throughout, and no second full environment is needed. It is the default update strategy for a [Kubernetes Deployment](./Kubernetes%20Deployment.md).

[Kubernetes](./Kubernetes.md) controls the pace with two settings. `maxSurge` is how many extra [Pods](./Pod.md) may exist above the desired count, and `maxUnavailable` is how many may be missing. With 4 replicas, `maxSurge: 1`, and `maxUnavailable: 0`:

| Step | Old Pods | New Pods | What happens                         |
| ---- | -------- | -------- | ------------------------------------ |
| 1    | 4        | 1        | One new Pod starts                   |
| 2    | 3        | 1        | It becomes ready, one old Pod leaves |
| 3    | 3        | 2        | Next new Pod starts                  |
| 4    | 2        | 2        | It becomes ready, one old Pod leaves |
| 5    | 0        | 4        | The pattern repeats until complete   |

Two versions serve traffic at the same time during the rollout, which causes intermittent errors that appear only during deploys. A page from the new version calls an API endpoint, the call lands on an old Pod that lacks it, and the request fails until a refresh. Each release must stay compatible with the one before it, which is the purpose of [expand and contract](./Expand%20and%20contract.md).

Two details prevent dropped requests. A [readiness probe](./Readiness%20probe.md) keeps traffic away from a new Pod until it can serve. Graceful shutdown lets an old Pod finish in-flight requests after `SIGTERM`, before the grace period (30 seconds by default) ends in `SIGKILL`.

A [rollback](./Rollback.md) is another rolling update in the opposite direction, so it takes time. For an instant switch, use a [blue-green deployment](./Blue-green%20deployment.md). To measure impact on a small share of traffic first, use a [canary release](./Canary%20release.md).

_Usage:_

"We get a handful of 404s from the API on every deploy, then they stop."

"During the rolling deployment, new frontends hit old backends. Ship the endpoint first, then the frontend that calls it."
