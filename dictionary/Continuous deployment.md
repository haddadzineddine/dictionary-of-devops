---
description: Automatically releasing every change that passes the pipeline to production, with no human approval step.
---

Continuous deployment is the practice of automatically releasing every change that passes the [pipeline](./Pipeline.md) to production, with no human approval step. It extends [continuous delivery](./Continuous%20delivery.md), where a person still chooses when to deploy. Under continuous deployment, merging to the main branch is the decision to ship.

Removing the human gate moves the responsibility for safety into automation. A bad change that the checks miss can reach users within minutes of being merged, so the safeguards have to exist before the gate is removed:

| Safeguard                | What it provides                                                     |
| ------------------------ | -------------------------------------------------------------------- |
| Reliable automated tests | Confidence that a green run means the change works                   |
| [Canary release](./Canary%20release.md) | Exposes a small share of traffic to the change first  |
| Health-based promotion   | Stops the rollout when error rate or latency worsens                 |
| Automated [rollback](./Rollback.md) | Returns to the previous version without waiting for a person |
| [Feature flag](./Feature%20flag.md) | Ships code switched off and enables it separately        |

Feature flags matter here because they separate deploying from releasing. Code can reach production on every merge while the behaviour stays off until product, support, and documentation are ready. Without flags, every merged change becomes visible to users, and developers start holding back merges, which recreates long-lived branches.

Database changes need particular care. An automated deploy will run a schema change as readily as a code change, and a destructive migration cannot be undone by redeploying old code. Teams use [expand and contract](./Expand%20and%20contract.md) so each deploy stays compatible with the version before it.

Continuous deployment does not suit every product. Mobile apps pass through store review, firmware is hard to update once shipped, and some regulated environments require a recorded human approval. Those teams usually stop at continuous delivery.

_Usage:_

"Who approved this going to production? I only merged it ten minutes ago."

"Nobody has to. We use continuous deployment, so the merge was the approval. If the canary's error rate had climbed, it would have rolled itself back."
