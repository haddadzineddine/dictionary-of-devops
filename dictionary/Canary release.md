---
description: A release strategy that sends a small share of real traffic to a new version and widens it only if metrics stay healthy.
---

A canary release is a release strategy that sends a small share of real production traffic to a new version, compares its behaviour with the current version, and increases the share only if it stays healthy. If it degrades, traffic returns to the old version, so the failure affects a small group of users.

A typical progression looks like this, though the percentages and timings vary:

| Stage | Traffic to new version | Hold       | Promote if                         |
| ----- | ---------------------- | ---------- | ---------------------------------- |
| 1     | 1%                     | 10 minutes | Errors and latency match baseline  |
| 2     | 10%                    | 30 minutes | Same, plus key business metrics    |
| 3     | 50%                    | 30 minutes | Same                               |
| 4     | 100%                   | Done       | Old version removed                |

A canary catches problems that tests miss, because real traffic has data, clients, and load patterns that staging lacks. It only helps if someone defines what "healthy" means before the release. Compare the canary with a baseline running the old version at the same moment, because comparing against yesterday's numbers mixes in normal traffic changes. Tools such as Argo Rollouts and Flagger can automate promotion and [rollback](./Rollback.md) based on these metrics.

Traffic splitting needs some care. A plain [Kubernetes Service](./Kubernetes%20Service.md) spreads connections across [Pods](./Pod.md), so a percentage can only be approximated by the replica ratio, such as one canary in ten Pods. Exact weights need an [Ingress](./Ingress.md) controller or [service mesh](./Service%20mesh.md) that supports them.

Small canaries have blind spots. At 1% on a quiet service, there may be too few requests for a problem to show up, and slow failures such as memory leaks can outlast the hold time. Assign users consistently, so that one person does not bounce between versions. As in a [rolling deployment](./Rolling%20deployment.md), both versions must work with the same database.

_Usage:_

"Should we ship the new ranking model to everyone tonight?"

"Do a canary release at 5% first and compare error rate and checkout conversion against the baseline for an hour."
