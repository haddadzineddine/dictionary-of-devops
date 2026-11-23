---
description: Writing security and compliance rules as executable code that automatically checks and blocks infrastructure changes.
---

Policy as code is the practice of writing organisational rules as machine-readable code, so that tools can evaluate changes against them automatically. Rules such as "no public storage buckets", "[containers](./Container.md) must not run as root", or "[images](./Container%20image.md) must come from our registry" are versioned, reviewed, and tested in the same way as application code. Open Policy Agent (with its Rego language), Kyverno, and HashiCorp Sentinel are examples.

Written guidelines depend on someone remembering them during review. A policy engine gives the same answer every time and reports which rule failed.

The same rule can be enforced at several points:

| Enforcement point         | What is checked                         | Effect of a violation            |
| ------------------------- | --------------------------------------- | -------------------------------- |
| Developer machine or PR   | Manifests and config files              | Early feedback                   |
| [Pipeline](./Pipeline.md) | A [Terraform](./Terraform.md) plan or built image | Blocks the merge or deploy |
| [Kubernetes](./Kubernetes.md) admission | Objects sent to the API server          | The API request is rejected      |
| Runtime audit             | Resources that already exist            | Reports or remediates            |

Earlier checks give faster feedback, while admission control is harder to bypass. An admission controller is a [control plane](./Control%20plane.md) plugin that can inspect or modify a request before it is stored. Policy engines usually attach as admission webhooks, so a blocked deploy appears as an error such as `admission webhook "..." denied the request`.

A new rule in blocking mode can break existing deployments. Run it in audit mode first, fix or exempt what it would reject, then enforce it. Each policy should tell the developer how to comply.

Admission webhooks also add an availability risk. If a webhook is set to `failurePolicy: Fail` and its service goes down, every matching API request is refused, which can block new [Pods](./Pod.md), including the replacement for the webhook itself. Run it with multiple replicas, and exclude system namespaces.

_Usage:_

"My deploy just got rejected by an admission webhook."

"That's policy as code. The rule says containers can't run as root, so set `runAsNonRoot` and apply it again."
