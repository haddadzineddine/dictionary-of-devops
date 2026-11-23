---
description: Running new code in production against real traffic or data, while users continue to see only the existing behaviour.
---

A dark launch is the release of new code to production so that it runs against real traffic or data while users continue to see the existing behaviour. The new path does its work, but its output is discarded or logged for comparison. This tests performance and correctness under production conditions before anyone depends on the result.

A common form is shadow traffic. The service handles each request as usual and also sends a copy to the new implementation asynchronously. The user receives the old response, while engineers compare the two responses and watch the new system's latency and error rate. A [feature flag](./Feature%20flag.md) often controls which requests are mirrored, and a [service mesh](./Service%20mesh.md) can mirror them at the network level.

Dark launches are often confused with neighbouring strategies:

| Strategy                                | Runs in production? | Do users see the result? |
| --------------------------------------- | ------------------- | ------------------------ |
| Dark launch                             | Yes                 | No                       |
| [Canary release](./Canary%20release.md) | Yes                 | A small share of users   |
| Feature flag rollout                    | Yes                 | Chosen users             |

The technique suits risky backend replacements, such as a rewritten pricing engine or a new search index, because staging rarely reproduces real data shapes and volume.

Side effects are the main hazard. If a mirrored request sends an email, charges a card, or writes to a shared database, it happens twice, and customers see duplicates from a feature meant to be invisible. Stub out writes and external calls in the dark path, or point them at isolated storage.

Dark traffic is also real load, and extra calls to a shared dependency can slow the live path. Sample a small percentage first, send copies with strict timeouts, and keep a way to turn mirroring off. Remove the comparison code after launch.

_Usage:_

"How do we know the new pricing engine handles real orders?"

"Dark launch it. Mirror 10% of requests, discard its output, and diff its prices against the old engine for a week."
