---
description: Storing, distributing, rotating, and auditing credentials such as passwords, API keys, and certificates.
---

Secrets management is the practice of controlling credentials such as passwords, API keys, tokens, and certificates throughout their lifecycle. It covers storage, access, delivery to applications, rotation (replacing them with new values), and auditing. HashiCorp Vault, AWS Secrets Manager, and Google Secret Manager are examples of dedicated stores.

The typical failure is a credential committed to Git. Deleting it in a later commit does not help, because it remains in the repository's history and in every clone. Once a secret has been pushed, treat it as compromised: revoke it, issue a new one, and check the access logs. Secret scanners in pre-commit hooks and [pipelines](./Pipeline.md) can catch many of these before they land.

Approaches form a ladder:

| Level | Approach                     | Remaining weakness                          |
| ----- | ---------------------------- | ------------------------------------------- |
| 1     | Hardcoded in source          | Visible to anyone with the code             |
| 2     | `.env` files on servers      | Copied around, rarely rotated               |
| 3     | [CI](./Continuous%20integration.md) or platform secret store | Long-lived, limited audit                   |
| 4     | [Kubernetes Secret](./Kubernetes%20Secret.md), encrypted at rest | Readable by anyone who can create [Pods](./Pod.md) in the namespace |
| 5     | Central secrets manager      | Applications still hold static values       |
| 6     | Short-lived dynamic credentials | Depends on the issuer being available    |

Dynamic credentials are issued on demand and expire automatically, so a leaked value has a short useful life. With workload identity or OIDC federation (trading a signed identity token for temporary access), a CI job or Pod receives temporary cloud permissions without a stored credential.

Rotation fails when nobody knows which services use a key, so the first attempt causes an outage. Automate it, and support two valid credentials during the changeover.

Secrets also leak through less obvious channels: logs, error reports, [Terraform state](./Terraform%20state.md), and [image layers](./Image%20layer.md). Inject values at runtime, and keep them out of builds and output.

_Usage:_

"I removed the leaked AWS key in the next commit, so we're fine, right?"

"No, it's still in history. Revoke it now. Then move it into secrets management."
