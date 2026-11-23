---
description: The gap that opens when live infrastructure no longer matches its declared configuration, often after manual changes.
---

Configuration drift is the difference that develops between what your configuration declares and what is actually running. It usually starts with a reasonable action: during an incident, someone opens a port in the cloud console, edits a [Kubernetes Deployment](./Kubernetes%20Deployment.md) with `kubectl edit`, or changes a setting on one server over SSH. The fix works, and nobody updates the code.

Drift has two recognisable consequences. The first is a hotfix that disappears. Weeks later, someone runs a routine [Terraform](./Terraform.md) apply, the tool restores the declared value, and the incident returns. The second is environments that behave differently for reasons no one can find, where a bug appears in production and cannot be reproduced in staging because their settings are no longer alike.

Detection depends on the layer:

| Layer          | How to detect drift                                                     |
| -------------- | ----------------------------------------------------------------------- |
| Cloud resources | A scheduled `terraform plan`, alerting on any non-empty result         |
| [Kubernetes](./Kubernetes.md) | A [GitOps](./GitOps.md) controller reporting an out-of-sync state       |
| Servers        | A [configuration management](./Configuration%20management.md) run in check mode |

Once found, each difference needs a decision:

| Decision | When                                     | Action                                  |
| -------- | ---------------------------------------- | --------------------------------------- |
| Adopt    | The live change is correct               | Update the code to match                |
| Revert   | The live change was a mistake            | Apply the code                          |
| Ignore   | Something else legitimately manages it   | Exclude the field, such as with `ignore_changes` |

Some differences are expected. A [Horizontal Pod Autoscaler](./Horizontal%20Pod%20Autoscaler.md) changes replica counts by design, so the replica field should not be owned by the manifest as well.

To prevent drift, restrict write access to production, keep a fast emergency route through the normal [pipeline](./Pipeline.md), and require a follow-up commit after any manual fix. [Immutable infrastructure](./Immutable%20infrastructure.md) reduces drift on servers because they are replaced instead of edited.

_Usage:_

"The rate limit we raised during Friday's incident is back to the old value."

"That was a console change, so it was configuration drift. Today's apply reverted it. Put the new value in the Terraform code."
