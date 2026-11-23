---
description: The packaged output of a build, such as a binary, archive, or container image, that is versioned and deployed.
---

A build artifact is the packaged output of a build: the file or bundle that actually gets deployed. Depending on the stack, it might be a compiled binary, a Java JAR, a zip of bundled JavaScript, a Python wheel, or a [container image](./Container%20image.md). The artifact is produced once by the [pipeline](./Pipeline.md), given a version, and stored in an [artifact registry](./Artifact%20registry.md).

The rule that follows is "build once, deploy many". The same artifact moves from test to staging to production, and only its configuration changes between environments. If each environment builds from source separately, the builds can differ. A dependency range may resolve to a newer release, a base image may have changed, or the compiler on one runner may differ. This produces a confusing failure: staging passed, production is broken, and the source commit is identical in both.

| Approach                 | What staging tested        | What production runs       |
| ------------------------ | -------------------------- | -------------------------- |
| Rebuild per environment  | Build A from commit `abc`  | Build B from commit `abc`  |
| Promote one artifact     | Build A from commit `abc`  | Build A from commit `abc`  |

Keeping one artifact across environments has consequences for how it is built. Environment-specific values such as database URLs must be supplied at runtime through environment variables or mounted files. Credentials must not be baked in, because anyone who can download the artifact could read them.

An artifact should be traceable. Record the commit it came from, its version, and a checksum or digest (a hash of its contents) so you can prove that what is running is what was tested. An [SBOM](./SBOM.md) can be published alongside it to list what it contains. Keep previous artifacts available, because a fast [rollback](./Rollback.md) depends on redeploying an earlier one instead of rebuilding it under pressure.

_Usage:_

"Staging is fine but prod throws an import error on the same commit."

"Are we rebuilding for each environment? Promote the build artifact that passed staging instead, so a dependency can't shift between the two."
