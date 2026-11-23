---
description: A MAJOR.MINOR.PATCH version format where each number signals whether a release breaks, extends, or fixes the public API.
---

Semantic versioning (SemVer) is a version number format, `MAJOR.MINOR.PATCH`, where each part tells consumers what kind of change a release contains. It applies to a declared public API, meaning whatever other code is expected to depend on.

| Part  | Example           | Increment when                          | Safe to adopt automatically? |
| ----- | ----------------- | --------------------------------------- | ---------------------------- |
| MAJOR | `1.4.2` → `2.0.0` | A change breaks existing consumers      | No, read the release notes   |
| MINOR | `1.4.2` → `1.5.0` | Functionality is added compatibly       | Usually                      |
| PATCH | `1.4.2` → `1.4.3` | A bug is fixed compatibly               | Usually                      |

Incrementing a number resets those to its right. Two suffixes extend the format. A pre-release such as `2.0.0-rc.1` sorts before `2.0.0`. Build metadata such as `2.0.0+abc123` is ignored when comparing versions.

Version ranges in package managers rely on these promises. In npm, `^1.4.2` accepts anything below `2.0.0`, and `~1.4.2` accepts anything below `1.5.0`. Versions below `1.0.0` are treated differently, because the specification says anything may change during initial development. For that reason `^0.4.2` only accepts `0.4.x`.

SemVer is a statement of intent by the maintainer, and no tool enforces it. A library can publish a "patch" that changes behaviour you relied on, and your build breaks on a Monday morning with no change in your own code. Sometimes the maintainer considers the old behaviour a bug, while you considered it a feature. This is the reason to commit a lockfile and use [dependency pinning](./Dependency%20pinning.md), so that upgrades happen deliberately and go through [continuous integration](./Continuous%20integration.md).

Applications that have no consumers of their API get less value from SemVer. A deployed web service is often better identified by a commit hash or build number, and SemVer is kept for libraries, APIs, and [Helm](./Helm.md) charts.

_Usage:_

"It's only a patch bump, so I merged it without testing."

"Semantic versioning is a promise, not a guarantee. That patch changed the date parser and broke invoices, so let the upgrade run through CI first."
