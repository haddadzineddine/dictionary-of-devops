---
description: A runtime switch that turns code paths on or off without a deploy, separating shipping code from releasing behaviour.
---

A feature flag is a runtime switch that turns a code path on or off without a new deployment. The code ships to production inside a conditional, and a configuration value or flag service decides who gets which path. This separates deploying code from releasing behaviour.

```ts
if (flags.isEnabled("new-checkout", { userId })) {
  return renderNewCheckout();
}
return renderOldCheckout();
```

Flags come in several kinds, with different lifespans:

| Type         | Purpose                                     | Expected lifespan |
| ------------ | ------------------------------------------- | ----------------- |
| Release      | Hide unfinished or unreleased work          | Days to weeks     |
| Experiment   | Compare variants in an A/B test             | Weeks             |
| Operational  | Kill switch to disable a costly feature     | Long-term         |
| Permission   | Enable features for a plan or customer      | Long-term         |

Release flags make [trunk-based development](./Trunk-based%20development.md) practical, because incomplete work can merge while switched off. They also allow a gradual rollout by user percentage, and turning a flag off is usually much faster than a [rollback](./Rollback.md) because nothing is rebuilt or redeployed. A [dark launch](./Dark%20launch.md) often relies on one.

The cost is accumulated flags. Each boolean doubles the possible code paths in principle, and nobody tests every combination. A common incident is a flag that has been "on" for two years: someone removes it from the flag service, the default evaluates to false, and production falls back to an old code path that no longer works. Give each release flag an owner and a removal date, and delete the conditional once the rollout is complete.

Decide what happens when the flag service is unreachable. Clients should cache the last known values and define a safe default for each flag, so that an outage in the flag service does not change application behaviour.

_Usage:_

"The new search is causing timeouts. Do we need to roll back?"

"No, it's behind a feature flag. Turn it off for everyone now, and we can look at the query tomorrow."
