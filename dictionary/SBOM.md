---
description: A software bill of materials, a machine-readable inventory of the components and versions inside a piece of software.
---

An SBOM (software bill of materials) is a machine-readable inventory of the components inside a piece of software. For each package it typically records the name, version, supplier, license, a unique identifier, and how components depend on each other. It works like an ingredient list for a [build artifact](./Build%20artifact.md) or [container image](./Container%20image.md).

The practical case for one appears when a serious vulnerability is announced in a widely used library. The immediate question is "are we affected, and where?". Without an inventory, engineers search repositories by hand and still miss copies pulled in several levels down or bundled inside a vendor's image. With SBOMs stored for every release, the answer is a query across existing data.

Two standard formats are in common use:

| Format    | Maintained by | Origin and emphasis                       |
| --------- | ------------- | ----------------------------------------- |
| SPDX      | Linux Foundation | Began with license compliance          |
| CycloneDX | OWASP         | Began with application security           |

Tools such as Syft and Trivy can generate either format. Generate the SBOM in the [pipeline](./Pipeline.md) at build time, from the final artifact when possible, because a source repository does not show what the base image added. Store it with the artifact and sign it, so that it can be tied to one specific build as part of the [software supply chain](./Software%20supply%20chain.md).

An SBOM says a component is present, without saying whether the vulnerable function is ever called. A companion format, VEX (Vulnerability Exploitability eXchange), lets a supplier state whether a product is affected by a given flaw. Generators can also miss statically linked libraries, copied source files, and binaries downloaded during a build.

The list of known vulnerabilities grows after a build, so stored SBOMs should be re-checked regularly, using the same vulnerability data as [image scanning](./Image%20scanning.md).

_Usage:_

"A critical CVE just dropped in that XML library. Do we ship it anywhere?"

"Query the SBOMs for every production release. That'll list the affected services without us grepping all the repos."
