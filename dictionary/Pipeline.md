---
description: An automated sequence of build, test, and release steps that runs when code changes, defined in a file stored with the code.
---

A pipeline is an automated sequence of steps that takes a code change from commit to a tested, packaged, and possibly deployed result. Pipelines are usually defined in a file stored with the code. A runner, which is a machine that executes the work, starts one when an event occurs, such as a push or a merged pull request.

| Level   | Meaning                         | Example                  |
| ------- | ------------------------------- | ------------------------ |
| Trigger | The event that starts a run     | Push to `main`           |
| Stage   | An ordered group of jobs        | Test                     |
| Job     | A set of steps on one runner    | Unit tests on Node 22    |
| Step    | A single command or action      | `npm test`               |

A minimal definition in GitHub Actions syntax:

```yaml
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
```

A typical pipeline supports [continuous integration](./Continuous%20integration.md) in its early stages, then produces a [build artifact](./Build%20artifact.md), publishes it to an [artifact registry](./Artifact%20registry.md), and deploys it through environments.

Slow pipelines are a common complaint. When a run takes forty minutes, developers batch up changes, stop watching results, and lose context before a failure comes back. The usual fixes are to run cheap checks first so failures appear early, cache dependencies between runs, split tests across parallel jobs, and run expensive suites only where they are relevant.

A job can pass locally and fail in the pipeline. Runners start from a clean environment, so anything that depended on a globally installed tool, a local file, or an unset environment variable breaks there. Treat that failure as information about a hidden dependency.

Pipelines often hold credentials that can publish packages or change production, which makes them an attack target in the [software supply chain](./Software%20supply%20chain.md). Limit each job's permissions, and do not expose secrets to runs triggered by untrusted pull requests.

_Usage:_

"The pipeline's been stuck on integration tests for half an hour again."

"Move lint and unit tests ahead of them so bad commits fail in two minutes, then split the integration suite across four jobs."
