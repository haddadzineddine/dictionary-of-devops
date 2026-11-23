---
description: The practice of merging work into a shared branch frequently, with an automated build and tests running on every change.
---

Continuous integration (CI) is the practice of merging every developer's work into a shared branch frequently, with an automated build and test run on each change. The term describes a team habit as well as tooling. A team with a CI server whose branches stay unmerged for weeks has automated tests, but it is not integrating continuously.

CI addresses the "works on my machine" problem. A change passes on a laptop with a particular runtime version, a leftover environment variable, and a cached dependency, then fails for everyone else. A CI run starts from a clean checkout in a defined environment, so hidden assumptions show up as failures within minutes of the push.

A [pipeline](./Pipeline.md) usually orders its checks so that fast ones fail first:

| Check             | What it catches                                    |
| ----------------- | -------------------------------------------------- |
| Lint and format   | Style problems and simple mistakes                 |
| Compile or build  | Syntax errors, type errors, missing imports        |
| Unit tests        | Broken logic in individual functions               |
| Integration tests | Components that fail when run together             |
| Packaging         | A [build artifact](./Build%20artifact.md) that cannot be produced |

Three habits keep CI useful. First, a failing main branch is fixed or reverted before other work continues, because everyone branching from it inherits the failure. Second, the run stays fast enough that people wait for the result instead of switching tasks. Third, flaky tests, which pass or fail at random on the same code, are fixed or quarantined. When a suite is flaky, people learn to press "re-run" until it goes green, and real failures start being dismissed as noise.

CI stops at verifying the change. Getting the verified change to users is the job of [continuous delivery](./Continuous%20delivery.md) or [continuous deployment](./Continuous%20deployment.md).

_Usage:_

"CI failed on my PR but the test passes locally."

"Check the Node version in the workflow. CI runs from a clean checkout, so it won't have whatever you installed globally."
