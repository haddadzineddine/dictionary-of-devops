---
description: Fixing dependencies to exact versions or hashes so that every build uses the same inputs and updates happen deliberately.
---

Dependency pinning is the practice of fixing each dependency to an exact version or content hash, so that a build uses the same inputs every time. Without it, a version range or moving tag can resolve to something new between two builds.

The familiar symptom is a build that passed on Friday and fails on Monday with no commits in between. A package several levels down published a new release, the range accepted it, and CI picked it up. [Semantic versioning](./Semantic%20versioning.md) ranges rely on maintainers classifying each change correctly, and on nobody publishing a malicious update.

Pinning has several levels of strength:

| Level          | Example                         | What it guarantees                      |
| -------------- | ------------------------------- | --------------------------------------- |
| Range          | `"express": "^4.18.0"`          | Compatible versions, by convention      |
| Exact version  | `"express": "4.18.2"`           | That package only, not its dependencies |
| Lockfile       | `package-lock.json`, `poetry.lock` | The full dependency tree              |
| Content hash   | `integrity: sha512-...`         | The exact bytes                         |

Commit the lockfile and install from it in CI. `npm ci`, for example, installs precisely what the lockfile says and fails if it disagrees with `package.json`, while `npm install` may update it.

The same rule applies beyond language packages. A [container image](./Container%20image.md) tag can be moved, so pin a base image by digest. Pin CI actions to a full commit SHA, because a tag such as `v4` can be repointed:

```dockerfile
FROM node:22-slim@sha256:<digest>
```

```yaml
- uses: actions/checkout@<full-commit-sha> # v4
```

Pinning protects the [software supply chain](./Software%20supply%20chain.md) from unexpected changes, but it also stops security fixes arriving automatically. Tools such as Renovate and Dependabot address this by opening regular pull requests that bump versions and run through [continuous integration](./Continuous%20integration.md). A short delay before adopting new releases gives others time to spot a bad one.

_Usage:_

"Nobody touched the repo over the weekend, but the build is red."

"The [Dockerfile](./Dockerfile.md) runs `npm install` with no lockfile, so a transitive package moved. Add dependency pinning and switch to `npm ci`."
