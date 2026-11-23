---
description: Returning a system to a previous known-good version after a bad release. It restores code, but not necessarily data.
---

A rollback is the act of returning a system to a previous known-good version after a release causes problems. It is usually a quick way to restore service, because it reuses something that has already run in production.

The available method depends on how the release was made:

| Method                  | How it works                                                    | Typical speed |
| ----------------------- | --------------------------------------------------------------- | ------------- |
| [Feature flag](./Feature%20flag.md) off | Disables the code path with no deploy           | Seconds       |
| Traffic switch          | Points back at the old [blue-green](./Blue-green%20deployment.md) environment | Seconds to minutes |
| Abort a [canary](./Canary%20release.md) | Returns the traffic share to the old version    | Seconds to minutes |
| Redeploy old artifact   | Rolls out the previous [build artifact](./Build%20artifact.md)  | Minutes       |
| `kubectl rollout undo`  | Restores the previous [Deployment](./Kubernetes%20Deployment.md) revision | Minutes |
| `git revert` with [GitOps](./GitOps.md) | Reverts the commit and lets the agent sync | Minutes       |

A rollback sometimes fails to fix the incident. The old code is running again and errors continue, because the release changed something other than code. Common causes:

- A database migration dropped or rewrote a column that the old version needs.
- New code already wrote data in a format the old code cannot read.
- A [ConfigMap](./ConfigMap.md), [Secret](./Kubernetes%20Secret.md), or infrastructure change is not versioned with the release.
- The real cause was never the release, such as a failing dependency or a traffic spike.
- The previous artifact was deleted by a retention policy.

Plan for this before release: keep schema changes backward compatible using [expand and contract](./Expand%20and%20contract.md), retain recent artifacts, and rehearse the procedure.

Rolling forward, meaning shipping a fix as a new version, can be safer when the fix is small and understood, or when data has changed so that old code cannot run.

_Usage:_

"We rolled back twenty minutes ago and checkout is still failing."

"Then it isn't only the code. That release ran a migration, so the rollback put old code on the new schema."
