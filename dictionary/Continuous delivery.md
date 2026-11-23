---
description: Keeping every change that passes the pipeline ready to release, with a human deciding when it goes to production.
---

Continuous delivery is the practice of keeping every change that passes the automated [pipeline](./Pipeline.md) in a state that can be released to production at any time. The final production deployment is a human decision, usually a button press or an approved promotion. The abbreviation CD is ambiguous because it also stands for [continuous deployment](./Continuous%20deployment.md), which removes that human step.

| Practice               | Build and test | Ready to release | Production deploy |
| ---------------------- | -------------- | ---------------- | ----------------- |
| [Continuous integration](./Continuous%20integration.md) | Automatic | Not guaranteed | Separate process |
| Continuous delivery    | Automatic      | Always           | Manual decision   |
| Continuous deployment  | Automatic      | Always           | Automatic         |

The problem it addresses is the large, stressful release. When a team ships every few months, each release bundles many unrelated changes, needs a code freeze, and often runs late into the evening. If production breaks, nobody can tell which of the many commits caused it. Shipping small batches means each release contains little, so a failure is easier to trace and to reverse with a [rollback](./Rollback.md).

Continuous delivery needs a few things in place. Each change produces one versioned [build artifact](./Build%20artifact.md), and that same artifact moves through test, staging, and production. Deployment is scripted, so the production deployment uses the steps already exercised in staging. Configuration lives outside the artifact, so one build can run in every environment.

The manual gate is useful where release timing is a business or compliance decision: a launch tied to an announcement, a required sign-off, or a customer maintenance window. The gate should be a decision only. If pressing the button also requires a runbook of manual steps, the team does not yet have continuous delivery.

_Usage:_

"Can we ship the pricing fix this afternoon, or do we wait for the release train?"

"We have continuous delivery now. Build 412 already passed staging, so someone only needs to approve the production step."
