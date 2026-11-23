---
description: Automatically checking a container image's packages against databases of known vulnerabilities.
---

Image scanning is the automated inspection of a [container image](./Container%20image.md) to find known security problems. A scanner identifies the operating system packages and application libraries in each [image layer](./Image%20layer.md), then compares their versions against vulnerability databases, reporting matches as CVEs (public identifiers for known flaws). Trivy, Grype, and the scanners built into many [container registries](./Container%20registry.md) are examples.

Scans can run at several points:

| Where                           | Purpose                                   |
| ------------------------------- | ----------------------------------------- |
| Developer machine               | Early feedback before pushing             |
| [Pipeline](./Pipeline.md)       | Block a build that fails the threshold    |
| Registry                        | Scan on push, and rescan stored images    |
| [Kubernetes](./Kubernetes.md) admission | Reject images without a passing result    |
| Running workloads               | Find newly disclosed issues in production |

Rescanning matters because an image that was clean last month can show a critical finding today, when a flaw is disclosed in a package it already contains.

Teams adopting a scanner usually meet alert fatigue first. The initial report on a full-distribution base image can list hundreds of findings, many with no available fix or in code the application never runs. If the pipeline blocks on all of them, developers learn to bypass it. A workable gate fails only on high or critical findings that have a fix available, with time-limited exceptions for the rest.

To reduce findings, use a slim or distroless base through a [multi-stage build](./Multi-stage%20build.md), and rebuild regularly. If the base uses [dependency pinning](./Dependency%20pinning.md), update the pin, or the rebuild picks up no fixes.

Linux distributions often backport fixes without changing the upstream version number, which can produce false positives in tools that ignore distribution advisories. A clean scan covers only known issues in detected components. It does not find flaws in your own code or zero-day vulnerabilities (flaws not yet public).

_Usage:_

"The scanner reports 312 CVEs and [CI](./Continuous%20integration.md) is blocked."

"Most are in the full Debian base. Move to a slim image, then have image scanning fail only on fixable criticals."
