---
description: Everything that goes into building and shipping software, including dependencies, tools, pipelines, images, and registries.
---

The software supply chain is everything involved in producing and delivering a piece of software: source code, third-party dependencies, base [images](./Container%20image.md), compilers, [pipelines](./Pipeline.md), registries, and deployment tools. Your application inherits the trust of each link. Installing one package can pull in many others that nobody on the team chose directly.

A supply chain attack compromises one of those links instead of the application. The malicious code then arrives through a normal, trusted update, with no vulnerability in your own code:

| Attack              | How it works                                                        | Defence                                    |
| ------------------- | ------------------------------------------------------------------- | ------------------------------------------ |
| Typosquatting       | Publishes a package with a name close to a popular one              | Review new dependencies                    |
| Dependency confusion | Public package shadows an internal name                            | Scoped names, a controlled [artifact registry](./Artifact%20registry.md) |
| Maintainer takeover | A trusted package gets a malicious release                          | [Dependency pinning](./Dependency%20pinning.md), delayed updates |
| Install scripts     | Package runs code during installation                               | Disable scripts where possible             |
| [CI](./Continuous%20integration.md) compromise | Attacker steals pipeline secrets or alters the build                | Least privilege, pinned CI actions         |
| Tampered artifact   | Image or package is altered after the build                         | Signing and digest verification            |

Defences work in layers. Know what you ship, using an [SBOM](./SBOM.md). Find known flaws with [image scanning](./Image%20scanning.md) and dependency audits. Control what enters the build with pinned versions and a proxy registry. Prove where artifacts came from through signing, with a tool such as Sigstore's cosign, and through provenance, which is a signed record of how and from what an artifact was built. The SLSA framework (Supply-chain Levels for Software Artifacts) defines progressive levels for build integrity.

A signature has no effect unless something checks it, so verify it at deploy time, for example with [policy as code](./Policy%20as%20code.md) in a [Kubernetes](./Kubernetes.md) admission controller.

_Usage:_

"Why does the build need network access to run a postinstall script from some transitive package?"

"It doesn't, and that's a software supply chain risk. Disable install scripts in CI and allowlist the few that need them."
