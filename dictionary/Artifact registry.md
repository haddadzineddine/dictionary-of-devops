---
description: A versioned store for build outputs and packages that pipelines publish to and deployments pull from.
---

An artifact registry is a server that stores versioned build outputs and packages, so that [pipelines](./Pipeline.md) can publish to it and deployments can pull from it. It may hold npm packages, Maven JARs, Python wheels, [Helm](./Helm.md) charts, and [container images](./Container%20image.md). JFrog Artifactory, Sonatype Nexus, and GitHub Packages are examples. A [container registry](./Container%20registry.md) is the same idea specialised for images.

Registries commonly serve three kinds of repository:

| Type    | Contains                                      | Used for                                   |
| ------- | --------------------------------------------- | ------------------------------------------ |
| Hosted  | Your own [build artifacts](./Build%20artifact.md) | Publishing internal packages and releases |
| Proxy   | Cached copies from a public registry          | Faster builds, tolerance of upstream outages |
| Virtual | One URL combining hosted and proxy            | Simple client configuration                |

A proxy repository addresses a failure many teams meet once. The public package registry has an outage, or a maintainer removes a package, and every build fails even though nobody changed any code. With a proxy in front, versions fetched earlier are still served from the cache.

A proxy also needs careful configuration. If a build looks up an internal package name and the registry also checks public sources, an attacker can publish a package with the same name and a higher version number. This is called dependency confusion, and it is a [software supply chain](./Software%20supply%20chain.md) risk. Scope internal names so that they resolve only to the hosted repository.

Published versions should be immutable. If `2.3.1` can be overwritten, two deployments of the same version can contain different code, and any recorded checksum stops matching. Retention rules need similar thought. Storage grows quickly, so old versions get cleaned up, but a policy that deletes by age can remove the version you need for a [rollback](./Rollback.md). Exclude anything currently deployed or recently released.

_Usage:_

"The rollback failed with a 404 on the old package."

"The artifact registry's cleanup job deleted everything older than 30 days. Change the policy to keep the last ten releases regardless of age."
