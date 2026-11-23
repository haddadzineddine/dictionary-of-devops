# DevOps Dictionary

A plain-English dictionary of getting code to production: delivery pipelines, containers, Kubernetes, infrastructure as code, release strategies and supply-chain safety.

Why is the pod in CrashLoopBackOff? Why does Terraform want to destroy the database? Why did the rollback not fix it?

Each of these has a short explanation once you know the right words. Entries define the term, describe the failure it explains where there is one, and end with a short example of engineers using it.

---

## Table of contents

- **[Section 1 — Delivery](#section-1--delivery)**
  - [Trunk-based development](#trunk-based-development)
  - [Feature branch](#feature-branch)
  - [Continuous integration](#continuous-integration)
  - [Continuous delivery](#continuous-delivery)
  - [Continuous deployment](#continuous-deployment)
  - [Pipeline](#pipeline)
  - [Build artifact](#build-artifact)
  - [Artifact registry](#artifact-registry)
  - [Semantic versioning](#semantic-versioning)
  - [Feature flag](#feature-flag)
- **[Section 2 — Containers](#section-2--containers)**
  - [Container](#container)
  - [Container image](#container-image)
  - [Dockerfile](#dockerfile)
  - [Image layer](#image-layer)
  - [Multi-stage build](#multi-stage-build)
  - [Linux namespace](#linux-namespace)
  - [cgroup](#cgroup)
  - [Container registry](#container-registry)
- **[Section 3 — Kubernetes](#section-3--kubernetes)**
  - [Kubernetes](#kubernetes)
  - [Control plane](#control-plane)
  - [Pod](#pod)
  - [Kubernetes Deployment](#kubernetes-deployment)
  - [ReplicaSet](#replicaset)
  - [Kubernetes Service](#kubernetes-service)
  - [Ingress](#ingress)
  - [ConfigMap](#configmap)
  - [Kubernetes Secret](#kubernetes-secret)
  - [StatefulSet](#statefulset)
  - [DaemonSet](#daemonset)
  - [Liveness probe](#liveness-probe)
  - [Readiness probe](#readiness-probe)
  - [Resource requests and limits](#resource-requests-and-limits)
  - [OOMKilled](#oomkilled)
  - [CrashLoopBackOff](#crashloopbackoff)
  - [Horizontal Pod Autoscaler](#horizontal-pod-autoscaler)
  - [Helm](#helm)
  - [Kubernetes Operator](#kubernetes-operator)
  - [Service mesh](#service-mesh)
- **[Section 4 — Infrastructure as Code](#section-4--infrastructure-as-code)**
  - [Infrastructure as code](#infrastructure-as-code)
  - [Terraform](#terraform)
  - [Terraform state](#terraform-state)
  - [Configuration drift](#configuration-drift)
  - [Immutable infrastructure](#immutable-infrastructure)
  - [GitOps](#gitops)
  - [Configuration management](#configuration-management)
  - [Policy as code](#policy-as-code)
  - [Secrets management](#secrets-management)
- **[Section 5 — Release Strategies](#section-5--release-strategies)**
  - [Rolling deployment](#rolling-deployment)
  - [Blue-green deployment](#blue-green-deployment)
  - [Canary release](#canary-release)
  - [Dark launch](#dark-launch)
  - [Rollback](#rollback)
  - [Expand and contract](#expand-and-contract)
- **[Section 6 — Supply Chain](#section-6--supply-chain)**
  - [Software supply chain](#software-supply-chain)
  - [Dependency pinning](#dependency-pinning)
  - [SBOM](#sbom)
  - [Image scanning](#image-scanning)

## Section 1 — Delivery

### Trunk-based development

Trunk-based development is a branching model in which all developers merge small changes into one shared branch, called the trunk (usually `main`), at least once a day. Work happens directly on the trunk or on a [feature branch](#feature-branch) that lives for a day or two. The trunk is kept in a state that could be released at any time.

The model addresses a familiar failure. Two developers work separately for three weeks, each with passing tests, then spend days resolving conflicts because both changed the same modules. Each branch worked alone, but the combination was never tested until the merge. Merging daily keeps each conflict small, and it makes [continuous integration](#continuous-integration) possible because there is one line of code to integrate against.

| Aspect          | Long-lived branches           | Trunk-based development       |
| --------------- | ----------------------------- | ----------------------------- |
| Branch lifetime | Weeks or months               | Hours to a couple of days     |
| Merge size      | Large                         | Small                         |
| Conflicts       | Found late, in bulk           | Found early, one at a time    |
| Unfinished work | Hidden on the branch          | Merged, but switched off      |

Unfinished work is merged behind a [feature flag](#feature-flag), which keeps the code inactive in production. Another option is branch by abstraction, where a new implementation is built behind an existing interface and swapped in once ready. Database changes follow [expand and contract](#expand-and-contract) for the same reason: each small step must be safe to ship alone.

The model depends on supporting habits. A fast automated [pipeline](#pipeline) must run on every change, reviews must happen within hours, and a broken trunk must be fixed or reverted before other work lands. Without those habits, frequent merges spread breakage to everyone.

_Usage:_

"My branch is three weeks old and the rebase has forty conflicts."

"That's what trunk-based development avoids. Split it up, merge the refactor today, and put the new checkout behind a flag."

### Feature branch

A feature branch is a Git branch created from the main branch to hold one change, such as a new endpoint or a bug fix, until it is reviewed and merged back. It lets a developer commit unfinished work without affecting anyone else, and it gives reviewers a single diff to read, usually through a pull request (a request to review and merge a branch).

| Step | Who        | What happens                                                        |
| ---- | ---------- | ------------------------------------------------------------------- |
| 1    | Developer  | Creates the branch from the latest main                             |
| 2    | Developer  | Commits and pushes the change                                       |
| 3    | [CI](#continuous-integration) | Builds and tests the branch                |
| 4    | Reviewer   | Reads the diff and requests changes or approves                     |
| 5    | Developer  | Updates from main and resolves any conflicts                        |
| 6    | Merge tool | Merges the change and deletes the branch                            |

The common problem is a branch that lives too long. Main moves on while the branch stands still, so the diff grows, reviewers put it off, and the final merge brings conflicts that are tedious and easy to get wrong.

A branch can also pass every check and still break main after merging, because the checks ran against an older copy of main. This is a semantic conflict: Git merges the text cleanly, but one side renamed a function the other side started calling.

Two practices reduce the risk. Keeping branches to a day or two, as in [trunk-based development](#trunk-based-development), limits how far they can drift. A merge queue, which tests each pull request against the latest main plus the changes ahead of it before merging, catches semantic conflicts before they land. Large work can be split into small merges by hiding incomplete behaviour behind a [feature flag](#feature-flag).

_Usage:_

"Both PRs were green, so why is main failing?"

"Your feature branch was tested before Sam's rename landed. Git merged them without a conflict, but your code still calls the old function. We should turn on the merge queue."

### Continuous integration

Continuous integration (CI) is the practice of merging every developer's work into a shared branch frequently, with an automated build and test run on each change. The term describes a team habit as well as tooling. A team with a CI server whose branches stay unmerged for weeks has automated tests, but it is not integrating continuously.

CI addresses the "works on my machine" problem. A change passes on a laptop with a particular runtime version, a leftover environment variable, and a cached dependency, then fails for everyone else. A CI run starts from a clean checkout in a defined environment, so hidden assumptions show up as failures within minutes of the push.

A [pipeline](#pipeline) usually orders its checks so that fast ones fail first:

| Check             | What it catches                                    |
| ----------------- | -------------------------------------------------- |
| Lint and format   | Style problems and simple mistakes                 |
| Compile or build  | Syntax errors, type errors, missing imports        |
| Unit tests        | Broken logic in individual functions               |
| Integration tests | Components that fail when run together             |
| Packaging         | A [build artifact](#build-artifact) that cannot be produced |

Three habits keep CI useful. First, a failing main branch is fixed or reverted before other work continues, because everyone branching from it inherits the failure. Second, the run stays fast enough that people wait for the result instead of switching tasks. Third, flaky tests, which pass or fail at random on the same code, are fixed or quarantined. When a suite is flaky, people learn to press "re-run" until it goes green, and real failures start being dismissed as noise.

CI stops at verifying the change. Getting the verified change to users is the job of [continuous delivery](#continuous-delivery) or [continuous deployment](#continuous-deployment).

_Usage:_

"CI failed on my PR but the test passes locally."

"Check the Node version in the workflow. CI runs from a clean checkout, so it won't have whatever you installed globally."

### Continuous delivery

Continuous delivery is the practice of keeping every change that passes the automated [pipeline](#pipeline) in a state that can be released to production at any time. The final production deployment is a human decision, usually a button press or an approved promotion. The abbreviation CD is ambiguous because it also stands for [continuous deployment](#continuous-deployment), which removes that human step.

| Practice               | Build and test | Ready to release | Production deploy |
| ---------------------- | -------------- | ---------------- | ----------------- |
| [Continuous integration](#continuous-integration) | Automatic | Not guaranteed | Separate process |
| Continuous delivery    | Automatic      | Always           | Manual decision   |
| Continuous deployment  | Automatic      | Always           | Automatic         |

The problem it addresses is the large, stressful release. When a team ships every few months, each release bundles many unrelated changes, needs a code freeze, and often runs late into the evening. If production breaks, nobody can tell which of the many commits caused it. Shipping small batches means each release contains little, so a failure is easier to trace and to reverse with a [rollback](#rollback).

Continuous delivery needs a few things in place. Each change produces one versioned [build artifact](#build-artifact), and that same artifact moves through test, staging, and production. Deployment is scripted, so the production deployment uses the steps already exercised in staging. Configuration lives outside the artifact, so one build can run in every environment.

The manual gate is useful where release timing is a business or compliance decision: a launch tied to an announcement, a required sign-off, or a customer maintenance window. The gate should be a decision only. If pressing the button also requires a runbook of manual steps, the team does not yet have continuous delivery.

_Usage:_

"Can we ship the pricing fix this afternoon, or do we wait for the release train?"

"We have continuous delivery now. Build 412 already passed staging, so someone only needs to approve the production step."

### Continuous deployment

Continuous deployment is the practice of automatically releasing every change that passes the [pipeline](#pipeline) to production, with no human approval step. It extends [continuous delivery](#continuous-delivery), where a person still chooses when to deploy. Under continuous deployment, merging to the main branch is the decision to ship.

Removing the human gate moves the responsibility for safety into automation. A bad change that the checks miss can reach users within minutes of being merged, so the safeguards have to exist before the gate is removed:

| Safeguard                | What it provides                                                     |
| ------------------------ | -------------------------------------------------------------------- |
| Reliable automated tests | Confidence that a green run means the change works                   |
| [Canary release](#canary-release) | Exposes a small share of traffic to the change first  |
| Health-based promotion   | Stops the rollout when error rate or latency worsens                 |
| Automated [rollback](#rollback) | Returns to the previous version without waiting for a person |
| [Feature flag](#feature-flag) | Ships code switched off and enables it separately        |

Feature flags matter here because they separate deploying from releasing. Code can reach production on every merge while the behaviour stays off until product, support, and documentation are ready. Without flags, every merged change becomes visible to users, and developers start holding back merges, which recreates long-lived branches.

Database changes need particular care. An automated deploy will run a schema change as readily as a code change, and a destructive migration cannot be undone by redeploying old code. Teams use [expand and contract](#expand-and-contract) so each deploy stays compatible with the version before it.

Continuous deployment does not suit every product. Mobile apps pass through store review, firmware is hard to update once shipped, and some regulated environments require a recorded human approval. Those teams usually stop at continuous delivery.

_Usage:_

"Who approved this going to production? I only merged it ten minutes ago."

"Nobody has to. We use continuous deployment, so the merge was the approval. If the canary's error rate had climbed, it would have rolled itself back."

### Pipeline

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

A typical pipeline supports [continuous integration](#continuous-integration) in its early stages, then produces a [build artifact](#build-artifact), publishes it to an [artifact registry](#artifact-registry), and deploys it through environments.

Slow pipelines are a common complaint. When a run takes forty minutes, developers batch up changes, stop watching results, and lose context before a failure comes back. The usual fixes are to run cheap checks first so failures appear early, cache dependencies between runs, split tests across parallel jobs, and run expensive suites only where they are relevant.

A job can pass locally and fail in the pipeline. Runners start from a clean environment, so anything that depended on a globally installed tool, a local file, or an unset environment variable breaks there. Treat that failure as information about a hidden dependency.

Pipelines often hold credentials that can publish packages or change production, which makes them an attack target in the [software supply chain](#software-supply-chain). Limit each job's permissions, and do not expose secrets to runs triggered by untrusted pull requests.

_Usage:_

"The pipeline's been stuck on integration tests for half an hour again."

"Move lint and unit tests ahead of them so bad commits fail in two minutes, then split the integration suite across four jobs."

### Build artifact

A build artifact is the packaged output of a build: the file or bundle that actually gets deployed. Depending on the stack, it might be a compiled binary, a Java JAR, a zip of bundled JavaScript, a Python wheel, or a [container image](#container-image). The artifact is produced once by the [pipeline](#pipeline), given a version, and stored in an [artifact registry](#artifact-registry).

The rule that follows is "build once, deploy many". The same artifact moves from test to staging to production, and only its configuration changes between environments. If each environment builds from source separately, the builds can differ. A dependency range may resolve to a newer release, a base image may have changed, or the compiler on one runner may differ. This produces a confusing failure: staging passed, production is broken, and the source commit is identical in both.

| Approach                 | What staging tested        | What production runs       |
| ------------------------ | -------------------------- | -------------------------- |
| Rebuild per environment  | Build A from commit `abc`  | Build B from commit `abc`  |
| Promote one artifact     | Build A from commit `abc`  | Build A from commit `abc`  |

Keeping one artifact across environments has consequences for how it is built. Environment-specific values such as database URLs must be supplied at runtime through environment variables or mounted files. Credentials must not be baked in, because anyone who can download the artifact could read them.

An artifact should be traceable. Record the commit it came from, its version, and a checksum or digest (a hash of its contents) so you can prove that what is running is what was tested. An [SBOM](#sbom) can be published alongside it to list what it contains. Keep previous artifacts available, because a fast [rollback](#rollback) depends on redeploying an earlier one instead of rebuilding it under pressure.

_Usage:_

"Staging is fine but prod throws an import error on the same commit."

"Are we rebuilding for each environment? Promote the build artifact that passed staging instead, so a dependency can't shift between the two."

### Artifact registry

An artifact registry is a server that stores versioned build outputs and packages, so that [pipelines](#pipeline) can publish to it and deployments can pull from it. It may hold npm packages, Maven JARs, Python wheels, [Helm](#helm) charts, and [container images](#container-image). JFrog Artifactory, Sonatype Nexus, and GitHub Packages are examples. A [container registry](#container-registry) is the same idea specialised for images.

Registries commonly serve three kinds of repository:

| Type    | Contains                                      | Used for                                   |
| ------- | --------------------------------------------- | ------------------------------------------ |
| Hosted  | Your own [build artifacts](#build-artifact) | Publishing internal packages and releases |
| Proxy   | Cached copies from a public registry          | Faster builds, tolerance of upstream outages |
| Virtual | One URL combining hosted and proxy            | Simple client configuration                |

A proxy repository addresses a failure many teams meet once. The public package registry has an outage, or a maintainer removes a package, and every build fails even though nobody changed any code. With a proxy in front, versions fetched earlier are still served from the cache.

A proxy also needs careful configuration. If a build looks up an internal package name and the registry also checks public sources, an attacker can publish a package with the same name and a higher version number. This is called dependency confusion, and it is a [software supply chain](#software-supply-chain) risk. Scope internal names so that they resolve only to the hosted repository.

Published versions should be immutable. If `2.3.1` can be overwritten, two deployments of the same version can contain different code, and any recorded checksum stops matching. Retention rules need similar thought. Storage grows quickly, so old versions get cleaned up, but a policy that deletes by age can remove the version you need for a [rollback](#rollback). Exclude anything currently deployed or recently released.

_Usage:_

"The rollback failed with a 404 on the old package."

"The artifact registry's cleanup job deleted everything older than 30 days. Change the policy to keep the last ten releases regardless of age."

### Semantic versioning

Semantic versioning (SemVer) is a version number format, `MAJOR.MINOR.PATCH`, where each part tells consumers what kind of change a release contains. It applies to a declared public API, meaning whatever other code is expected to depend on.

| Part  | Example           | Increment when                          | Safe to adopt automatically? |
| ----- | ----------------- | --------------------------------------- | ---------------------------- |
| MAJOR | `1.4.2` → `2.0.0` | A change breaks existing consumers      | No, read the release notes   |
| MINOR | `1.4.2` → `1.5.0` | Functionality is added compatibly       | Usually                      |
| PATCH | `1.4.2` → `1.4.3` | A bug is fixed compatibly               | Usually                      |

Incrementing a number resets those to its right. Two suffixes extend the format. A pre-release such as `2.0.0-rc.1` sorts before `2.0.0`. Build metadata such as `2.0.0+abc123` is ignored when comparing versions.

Version ranges in package managers rely on these promises. In npm, `^1.4.2` accepts anything below `2.0.0`, and `~1.4.2` accepts anything below `1.5.0`. Versions below `1.0.0` are treated differently, because the specification says anything may change during initial development. For that reason `^0.4.2` only accepts `0.4.x`.

SemVer is a statement of intent by the maintainer, and no tool enforces it. A library can publish a "patch" that changes behaviour you relied on, and your build breaks on a Monday morning with no change in your own code. Sometimes the maintainer considers the old behaviour a bug, while you considered it a feature. This is the reason to commit a lockfile and use [dependency pinning](#dependency-pinning), so that upgrades happen deliberately and go through [continuous integration](#continuous-integration).

Applications that have no consumers of their API get less value from SemVer. A deployed web service is often better identified by a commit hash or build number, and SemVer is kept for libraries, APIs, and [Helm](#helm) charts.

_Usage:_

"It's only a patch bump, so I merged it without testing."

"Semantic versioning is a promise, not a guarantee. That patch changed the date parser and broke invoices, so let the upgrade run through CI first."

### Feature flag

A feature flag is a runtime switch that turns a code path on or off without a new deployment. The code ships to production inside a conditional, and a configuration value or flag service decides who gets which path. This separates deploying code from releasing behaviour.

```ts
if (flags.isEnabled("new-checkout", { userId })) {
  return renderNewCheckout();
}
return renderOldCheckout();
```

Flags come in several kinds, with different lifespans:

| Type         | Purpose                                     | Expected lifespan |
| ------------ | ------------------------------------------- | ----------------- |
| Release      | Hide unfinished or unreleased work          | Days to weeks     |
| Experiment   | Compare variants in an A/B test             | Weeks             |
| Operational  | Kill switch to disable a costly feature     | Long-term         |
| Permission   | Enable features for a plan or customer      | Long-term         |

Release flags make [trunk-based development](#trunk-based-development) practical, because incomplete work can merge while switched off. They also allow a gradual rollout by user percentage, and turning a flag off is usually much faster than a [rollback](#rollback) because nothing is rebuilt or redeployed. A [dark launch](#dark-launch) often relies on one.

The cost is accumulated flags. Each boolean doubles the possible code paths in principle, and nobody tests every combination. A common incident is a flag that has been "on" for two years: someone removes it from the flag service, the default evaluates to false, and production falls back to an old code path that no longer works. Give each release flag an owner and a removal date, and delete the conditional once the rollout is complete.

Decide what happens when the flag service is unreachable. Clients should cache the last known values and define a safe default for each flag, so that an outage in the flag service does not change application behaviour.

_Usage:_

"The new search is causing timeouts. Do we need to roll back?"

"No, it's behind a feature flag. Turn it off for everyone now, and we can look at the query tomorrow."

## Section 2 — Containers

### Container

A container is a running process, or group of processes, that the Linux kernel isolates from the rest of the machine. It gets its own filesystem from a [container image](#container-image), its own view of the system through [Linux namespaces](#linux-namespace), and resource limits through [cgroups](#cgroup). It shares the host's kernel, which is the main difference from a virtual machine.

| Aspect    | Container                     | Virtual machine                       |
| --------- | ----------------------------- | ------------------------------------- |
| Kernel    | Shares the host's kernel      | Runs its own kernel                   |
| Boundary  | Kernel isolation features     | Hypervisor (software that runs VMs)   |
| Startup   | Starts a process              | Boots an operating system             |
| Footprint | Application and its libraries | Full operating system                 |

Sharing a kernel has practical effects. A Linux container needs a Linux kernel, so Docker Desktop on macOS and Windows runs one inside a lightweight virtual machine. A kernel vulnerability can affect every container on the host, so the isolation is weaker than a VM's. CPU architecture also still matters: an image built only for ARM fails on an x86 host with `exec format error` unless emulation is available.

A container lives only as long as its main process. When that process exits, the container stops, even if other processes were running inside it. If a script starts a background daemon and then finishes, the container exits immediately.

Containers are disposable, and their writable filesystem is deleted along with them. A common early mistake is writing uploads or a database file to the container's local disk, then finding the data gone after the next deploy replaced it. Data that must survive belongs in a mounted volume or an external service. Treating containers as replaceable is what makes [immutable infrastructure](#immutable-infrastructure) and [Kubernetes](#kubernetes) scheduling workable.

_Usage:_

"All the uploaded avatars disappeared after this morning's deploy."

"They were saved inside the container, so they went away when it was replaced. Put them in object storage or a mounted volume."

### Container image

A [container](#container) image is a read-only package containing everything needed to start one: a filesystem made of [image layers](#image-layer), plus metadata such as the default command, environment variables, working directory, and user. An image is to a container roughly what a program file is to a running process. One image can start many identical containers.

Images are usually built from a [Dockerfile](#dockerfile) and follow the OCI (Open Container Initiative) format, so an image built with one tool can run on other compatible runtimes. They are stored in a [container registry](#container-registry) and are a common form of [build artifact](#build-artifact).

An image can be referred to in two ways:

| Reference | Example                 | Can it change?                       |
| --------- | ----------------------- | ------------------------------------ |
| Tag       | `api:1.4.2`, `api:latest` | Yes, a tag can be moved to a new image |
| Digest    | `api@sha256:9f86d0...`  | No, it is a hash of the content      |

The difference causes a recognisable failure. Two servers both run `api:latest`, but they behave differently because each pulled the tag at a different time and got a different image. The name `latest` has no special meaning. It is only the default tag used when none is specified, and it does not guarantee the newest build. In production, use a unique version tag per build, or a digest when you need an exact guarantee, as described in [dependency pinning](#dependency-pinning).

One image name can cover several CPU architectures through a multi-platform index, a list pointing to one image per platform. If no entry matches the machine, the pull fails with a "no matching manifest" error.

Images accumulate known vulnerabilities over time as new flaws are disclosed. Rebuild them regularly on an updated base and use [image scanning](#image-scanning) to find what needs attention.

_Usage:_

"Two nodes are running the same image but only one has the bug."

"Same tag, or same digest? If we deployed `latest`, they may have pulled different builds. Compare the digests."

### Dockerfile

A Dockerfile is a text file of ordered instructions that a builder executes to produce a [container image](#container-image). Each instruction starts from the result of the previous one, and those that change files produce a new [image layer](#image-layer).

```dockerfile
FROM node:22-slim
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --omit=dev
COPY . .
USER node
CMD ["node", "server.js"]
```

| Instruction  | Runs at    | Purpose                                     |
| ------------ | ---------- | ------------------------------------------- |
| `FROM`       | Build      | Selects the base image                      |
| `COPY`       | Build      | Adds files from the build context           |
| `RUN`        | Build      | Executes a command and saves the result     |
| `ENV`        | Both       | Sets environment variables                  |
| `USER`       | Both       | Sets the user for later steps and at runtime |
| `ENTRYPOINT` | Start      | Sets the executable for the [container](#container) |
| `CMD`        | Start      | Sets default arguments or a default command |

Instruction order controls build speed. Builders reuse cached layers until they reach a step whose input changed, then rerun everything after it. If `COPY . .` comes before the dependency install, editing one source file reinstalls every package, and a build that should take seconds takes minutes. Copying the lockfile first, as above, keeps the install step cached until dependencies change.

Three other mistakes are common. Without a `.dockerignore` file, the build context can include `node_modules`, `.git`, and local `.env` files, and `COPY . .` puts them in the image. Passing a password through `ARG` or `ENV` can leave it readable in the image's metadata or history, so use the builder's secret mount feature. Containers run as root unless the image or runtime sets another user.

The form of `CMD` affects shutdown. `CMD node server.js` runs under a shell, which may not forward the stop signal to Node, so the container hangs until it is force-killed. The JSON form runs the process directly. To keep build tools out of the result, use a [multi-stage build](#multi-stage-build).

_Usage:_

"Every [CI](#continuous-integration) build reinstalls all our dependencies, even for a typo fix."

"Your Dockerfile copies the whole repo before `npm ci`. Copy the lockfile first so the install layer stays cached."

### Image layer

An image layer is a read-only set of filesystem changes: files added, modified, or deleted relative to the layer below it. A [container image](#container-image) is an ordered stack of these layers, and a union filesystem merges them into the single directory tree that a [container](#container) sees. In a [Dockerfile](#dockerfile), instructions such as `RUN`, `COPY`, and `ADD` each create a layer.

Layers are identified by a hash of their content, which gives three benefits:

| Benefit       | What happens                                                         |
| ------------- | -------------------------------------------------------------------- |
| Build caching | Unchanged steps reuse an existing layer                              |
| Faster pushes | A [container registry](#container-registry) receives only the layers it lacks |
| Shared storage | Images with the same base keep one copy on disk                     |

When a container starts, the runtime adds a thin writable layer on top. Changing an existing file copies it into that layer first (copy-on-write), and the layer is deleted with the container.

Because lower layers are read-only, deleting a file in a later layer does not remove it. The later layer records a "whiteout" marker that hides the file, while the bytes remain in the earlier layer. For example, a Dockerfile downloads a 500 MB archive in one `RUN`, deletes it in the next, and the image stays 500 MB larger.

The same mechanism creates a security problem. If a credential was copied in and removed later, anyone who can pull the image can extract the earlier layer and read it. Treat any secret that ever entered a layer as leaked, and rotate it.

To keep layers small, create and delete temporary files within the same `RUN` command, or use a [multi-stage build](#multi-stage-build). `docker history` shows what each layer added.

_Usage:_

"I added `rm -rf /tmp/build` at the end, but the image didn't shrink."

"That's a new image layer hiding the files. The bytes are still in the earlier one. Do the cleanup in the same `RUN` step."

### Multi-stage build

A multi-stage build is a [Dockerfile](#dockerfile) with more than one `FROM` instruction, where each `FROM` starts a separate stage and later stages copy only the files they need from earlier ones. Compilers, package managers, and source code stay in the build stage. The final [container image](#container-image) contains the application and its runtime needs.

```dockerfile
FROM golang:1.23 AS build
WORKDIR /src
COPY . .
RUN CGO_ENABLED=0 go build -o /app ./cmd/api

FROM gcr.io/distroless/static-debian12
COPY --from=build /app /app
ENTRYPOINT ["/app"]
```

Without this technique, a small service can ship as an image of more than a gigabyte because it includes the entire toolchain used to compile it. Large images are slower to push and pull, which slows deployments and the start of new replicas during scaling. Every extra package also adds findings to [image scanning](#image-scanning) and gives an attacker more tools to use after a break-in.

| Final base    | Contains                             | Trade-off                                |
| ------------- | ------------------------------------ | ---------------------------------------- |
| Full distro   | Package manager, shell, utilities    | Large, easy to debug                     |
| Slim          | Reduced distribution                 | Smaller, still has a shell               |
| Distroless    | Runtime libraries, no shell          | Small, harder to debug                   |
| `scratch`     | Nothing                              | Needs a fully static binary              |

Smaller bases can fail in less obvious ways. A dynamically linked binary copied into `scratch` fails with `no such file or directory`, even though the file is there, because the missing item is the system's dynamic linker. Binaries built against glibc can fail the same way on Alpine, which uses musl. Apps that make HTTPS calls may also need CA certificates copied in, or every TLS connection fails. Build statically, or choose a base that matches the build stage.

A build can stop at a named stage using `--target`, which is useful for running tests in the build environment. Layers from earlier stages are not part of the final image, although they can remain in the builder's cache.

_Usage:_

"Why is a 20 MB Go service shipping as a 1.2 GB image?"

"It still has the whole Go toolchain in it. Switch to a multi-stage build and copy only the binary into a distroless base."

### Linux namespace

A Linux namespace is a kernel feature that gives a group of processes their own isolated view of one system resource. A [container](#container) is largely an ordinary process placed in a set of these namespaces, with [cgroups](#cgroup) limiting what it can consume.

| Namespace | Isolates                         | Effect inside a container                    |
| --------- | -------------------------------- | -------------------------------------------- |
| PID       | Process IDs                      | Main process appears as PID 1                |
| Network   | Interfaces, IPs, ports, routes   | Has its own `localhost`                      |
| Mount     | Filesystem mount points          | Sees the [image](#container-image) as its root filesystem |
| UTS       | Hostname                         | Has its own hostname                         |
| IPC       | Shared memory, message queues    | Cannot reach other containers' queues        |
| User      | User and group IDs               | Root inside can map to non-root outside      |
| Cgroup    | View of the cgroup hierarchy     | Sees its own cgroup as the root              |
| Time      | Boot and monotonic clocks        | Can have offset clocks                       |

The network namespace explains a frequent beginner failure. An app in one container connects to a database at `localhost:5432` and gets "connection refused", although the database is running. `localhost` refers to the container's own namespace, where nothing listens on that port. Connect using the other container's service name instead. Inside a [Kubernetes](#kubernetes) [Pod](#pod), the reverse applies: containers share one network namespace, so they do reach each other on `localhost`.

The PID namespace explains slow shutdowns. The kernel treats PID 1 specially: a signal with no installed handler is ignored instead of taking its default action. An app that never registers a `SIGTERM` handler therefore keeps running when asked to stop, and the runtime force-kills it after a timeout. Add a handler, or run a small init process such as `tini` as PID 1.

Namespaces hide resources but do not make a process secure by themselves. All containers on a host still share one kernel.

_Usage:_

"The API can't reach Redis on localhost, but Redis is definitely up."

"Different containers, different network namespaces. `localhost` is the API container itself. Use the `redis` hostname."

### cgroup

A cgroup (control group) is a Linux kernel feature that limits and measures the resources used by a group of processes. Where a [Linux namespace](#linux-namespace) controls what a process can see, a cgroup controls how much it can use. [Container](#container) runtimes create a cgroup for each one, and the [resource requests and limits](#resource-requests-and-limits) set in [Kubernetes](#kubernetes) are enforced through them. Most current Linux distributions default to cgroup v2.

| Resource | Limit example                | What happens at the limit                     |
| -------- | ---------------------------- | --------------------------------------------- |
| CPU      | 0.5 CPU                      | Processes are paused until the next period    |
| Memory   | 512 MiB                      | Reclaim is attempted, then the OOM killer runs |
| PIDs     | 200 processes                | Creating another process or thread fails      |
| Block I/O | Bytes or operations per second | Reads and writes are slowed                 |

The two limits behave very differently. Exceeding a memory limit gets a process killed, which appears in Kubernetes as [OOMKilled](#oomkilled). Exceeding a CPU limit only slows the process down, which makes it harder to notice.

CPU limits work as a quota per scheduling period, by default 100 ms. A limit of 0.5 CPU allows 50 ms of CPU time per period, counted across all threads. Four busy threads spend that in about 12.5 ms, then all of them are paused for the remaining 87.5 ms. The result is a service with latency spikes while its average CPU usage looks moderate. The metric to check is CPU throttling, reported in the cgroup's `cpu.stat` file.

Software inside the container may not know about these limits. Tools such as `free` read `/proc/meminfo`, which reports the host's memory. An application that sizes its heap or thread pool from host numbers can overrun its limit. Many language runtimes now read cgroup limits, but check yours.

_Usage:_

"P99 latency jumps every few seconds, but the CPU graph sits at 40%."

"Check throttling. The cgroup quota could be used up early in each period, and the threads then sit paused until the next one."

### Container registry

A [container](#container) registry is a server that stores [container images](#container-image) and serves them to the machines that run them. A [pipeline](#pipeline) pushes a new image after a build, and a host or [Kubernetes](#kubernetes) node pulls it before starting a container. Docker Hub, GitHub Container Registry, Amazon ECR, and Harbor are examples. A general [artifact registry](#artifact-registry) can often store images alongside other package types.

An image reference names its location in the registry:

| Part       | Example         | Meaning                                  |
| ---------- | --------------- | ---------------------------------------- |
| Registry   | `ghcr.io`       | The server. Docker Hub if omitted        |
| Repository | `acme/api`      | A collection of related images           |
| Tag        | `1.4.2`         | A movable, human-readable label          |
| Digest     | `sha256:9f86...` | The immutable hash of the image manifest |

Together these form `ghcr.io/acme/api:1.4.2`. Pushes and pulls transfer only the [image layers](#image-layer) the other side lacks, so images that share a base move quickly.

Registry problems usually appear in Kubernetes as `ErrImagePull`, followed by `ImagePullBackOff` while the kubelet (the agent on each node) waits between retries. The usual causes are a mistyped name, a tag that was never pushed, missing credentials for a private repository, or a rate limit. Docker Hub limits pulls, so a cluster that scales up can hit the limit and fail to start workloads that have nothing wrong with them. A pull-through cache, which is a local registry that fetches and stores upstream images on first use, avoids that dependency.

Two settings are worth enabling. Tag immutability, offered by many registries, prevents an existing tag from being pushed over, so `1.4.2` always means the same image. A retention policy removes old images to control storage costs, but it should keep anything currently deployed and enough history for a [rollback](#rollback). Registries can also run [image scanning](#image-scanning) when an image is pushed.

_Usage:_

"The new [pods](#pod) are all stuck in ImagePullBackOff."

"Check that the tag exists in the container registry. If it does, the node's pull secret has probably expired."

## Section 3 — Kubernetes

### Kubernetes

Kubernetes (often written K8s) is an open-source system that runs [containers](#container) across a group of machines, called a cluster. You describe the state you want, such as "three copies of this [image](#container-image), reachable on this port", and Kubernetes starts, places, connects, and replaces workloads to keep that state true.

A cluster has two parts. The [control plane](#control-plane) stores the desired state and makes decisions. Worker machines, called nodes, run the workloads inside [Pods](#pod).

Most work happens through a small set of object types:

| Need                          | Object                                                    |
| ----------------------------- | --------------------------------------------------------- |
| Run a stateless app           | [Deployment](#kubernetes-deployment)                |
| Run an app with stable identity | [StatefulSet](#statefulset)                         |
| Run one copy per node         | [DaemonSet](#daemonset)                               |
| Give Pods a stable address    | [Service](#kubernetes-service)                      |
| Route outside HTTP traffic    | [Ingress](#ingress)                                   |
| Supply settings               | [ConfigMap](#configmap)                               |
| Supply credentials            | [Secret](#kubernetes-secret)                        |

The behaviour that defines Kubernetes is reconciliation. Controllers, which are background loops, compare the declared state with the actual state and act on any difference. For example, you delete a misbehaving Pod and a new one appears within seconds. You removed the instance but left the instruction that says it should exist. To stop it, scale down or delete the object that owns it.

Reconciliation also means that changes are asynchronous. A successful `kubectl apply` only records your intent in the API. It does not mean the app is running, so a deploy can "succeed" while every new Pod fails to start. Check the rollout status and Pod events afterwards.

Kubernetes adds real operational weight: networking, upgrades, access control, and many concepts to learn. A managed offering such as Amazon EKS, Google GKE, or Azure AKS runs the control plane for you, but your team still owns the workloads. A single small service often does not need it.

_Usage:_

"I keep deleting this broken pod and it keeps coming back."

"That's Kubernetes doing its job. The Deployment still asks for three replicas, so scale that down or fix the image."

### Control plane

The control plane is the set of [Kubernetes](#kubernetes) components that stores cluster state and decides what should run where. Worker nodes run the applications. The control plane tells them what to run and tracks the result.

| Component                | Role                                                                 |
| ------------------------ | -------------------------------------------------------------------- |
| API server               | Front door for all reads and writes, with authentication and validation |
| etcd                     | Consistent key-value store that holds all cluster state              |
| Scheduler                | Picks a node for each unscheduled [Pod](#pod)                    |
| Controller manager       | Runs the loops that reconcile actual state with desired state        |
| Cloud controller manager | Talks to the cloud provider for load balancers, routes, and nodes    |

Each worker node runs a kubelet, an agent that watches the API server and starts the Pod's [containers](#container) through a container runtime. Components communicate through the API server, and only the API server talks to etcd.

A typical sequence for creating a [Deployment](#kubernetes-deployment): the API server validates and stores it in etcd, a controller creates a [ReplicaSet](#replicaset) and Pods, the scheduler assigns each Pod to a node, and that node's kubelet starts it.

Control plane failure does not immediately stop applications. Containers on healthy nodes keep running and can keep serving traffic, while `kubectl` commands fail or time out. The damage is that nothing can change: no new deployments, no scaling, and no rescheduling of Pods from a node that dies.

etcd needs a majority of its members to accept writes, so production clusters run an odd number, usually three or five. Three members tolerate one failure, and five tolerate two. Losing etcd without a backup means losing the cluster's definition, so self-managed clusters need tested backups. Managed services such as EKS, GKE, and AKS operate the control plane for you.

_Usage:_

"kubectl is timing out. Is the site down?"

"No, the app's still serving. The control plane is having trouble, so running pods are fine, but hold off on deploys until it recovers."

### Pod

A Pod is the smallest unit [Kubernetes](#kubernetes) can deploy: one or more [containers](#container) scheduled together on the same node. Containers in a Pod share a network namespace, so they have one IP address and reach each other on `localhost`, and they can mount the same volumes. Most Pods hold one application container, sometimes with a sidecar that ships logs or an init container that runs setup before the app starts.

Pods are disposable. A Pod is bound to one node and is not moved. If the node fails, or the Pod is deleted, a controller such as a [Deployment](#kubernetes-deployment) creates a replacement with a new name and a new IP address. Code that stores a Pod IP in configuration breaks the next time that happens, so clients should connect through a [Service](#kubernetes-service).

A restart is different from a replacement. When a container crashes, the kubelet (the agent on each node) restarts that container inside the same Pod, which keeps its name and IP. Files in the container's writable layer are lost, while an `emptyDir` volume survives until the Pod is removed.

| Phase     | Meaning                                                        |
| --------- | -------------------------------------------------------------- |
| Pending   | Accepted, but not all containers have started yet              |
| Running   | Bound to a node, with at least one container running or starting |
| Succeeded | All containers exited successfully and will not restart        |
| Failed    | All containers stopped, and at least one failed                |
| Unknown   | The Pod's state cannot be obtained, often a node problem       |

A Pod stuck in Pending usually has no node that fits its [resource requests](#resource-requests-and-limits), or it is waiting on a volume. The Events section of `kubectl describe pod` gives the reason. Statuses such as [CrashLoopBackOff](#crashloopbackoff) appear in `kubectl get pods` but are not phases.

_Usage:_

"The payment service had the worker's IP hardcoded, and it broke after the node drain."

"Pod IPs change whenever a pod gets replaced. Point it at the Service name instead."

### Kubernetes Deployment

A [Kubernetes](#kubernetes) Deployment is an object that declares how a stateless application should run: which [container image](#container-image), how many replicas, and how to update them. It does not manage [Pods](#pod) directly. It creates a [ReplicaSet](#replicaset) for each version of the Pod template, and shifts replicas between them during an update.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 3
  selector:
    matchLabels: { app: api }
  template:
    metadata:
      labels: { app: api }
    spec:
      containers:
        - name: api
          image: ghcr.io/acme/api:1.4.2
```

Changing anything in `template`, such as the image tag, triggers a [rolling deployment](#rolling-deployment):

| Step | What happens                                                        |
| ---- | ------------------------------------------------------------------- |
| 1    | A new ReplicaSet is created for the new template                    |
| 2    | It scales up, within the `maxSurge` allowance                       |
| 3    | Old Pods are removed as new ones pass their [readiness probe](#readiness-probe), within `maxUnavailable` |
| 4    | The old ReplicaSet stays at zero replicas, available for [rollback](#rollback) |

Both `maxSurge` and `maxUnavailable` default to 25%. Changing only `replicas` scales the current ReplicaSet and does not start a rollout. Editing a [ConfigMap](#configmap) does not start one either, which is why a config change can appear to do nothing.

A failed rollout often looks like a hang. If new Pods never become ready, the update stops partway and old Pods keep serving, which protects users. After `progressDeadlineSeconds`, 600 by default, the Deployment reports `ProgressDeadlineExceeded`. It does not roll itself back. Someone must fix the image or run `kubectl rollout undo`. [Pipelines](#pipeline) should run `kubectl rollout status` so that a stuck rollout fails the job, because `kubectl apply` returns success as soon as the change is accepted.

Use a [StatefulSet](#statefulset) instead when replicas need stable identities or individual storage.

_Usage:_

"[CI](#continuous-integration) says the deploy passed, but we're still serving the old version."

"Apply only submits the change. The Kubernetes Deployment is stuck because the new pods are failing readiness. Check `kubectl rollout status`."

### ReplicaSet

A ReplicaSet is a [Kubernetes](#kubernetes) object that keeps a specified number of matching [Pods](#pod) running. If there are too few it creates more from its Pod template, and if there are too many it deletes the excess. It finds its Pods using a label selector, which is a query over key-value tags attached to objects.

You rarely create one by hand. A [Deployment](#kubernetes-deployment) creates and owns ReplicaSets, one per version of the Pod template. That is why Pod names have two generated parts: in `api-7d9f6c5b4-x2kqp`, `7d9f6c5b4` identifies the ReplicaSet and `x2kqp` identifies the Pod.

| Object     | Responsible for                              |
| ---------- | -------------------------------------------- |
| Deployment | Versions, rollout strategy, and history      |
| ReplicaSet | The replica count for one version            |
| Pod        | Running the [containers](#container)     |

During a [rolling deployment](#rolling-deployment), the Deployment scales a new ReplicaSet up and the old one down. Old ReplicaSets stay at zero replicas so that a [rollback](#rollback) can scale one back up. `revisionHistoryLimit`, 10 by default, sets how many are kept. This is why `kubectl get rs` shows a list of entries with zero Pods, and they are normally safe to leave.

Three behaviours cause confusion. Manually scaling a ReplicaSet owned by a Deployment does not last, because the Deployment sets it back to the declared count. Editing the template of a standalone ReplicaSet does not update running Pods, because the template is only used when creating new ones, so the old [image](#container-image) keeps running until those Pods are replaced. A ReplicaSet can also adopt existing Pods that have no owner and match its selector, so overlapping labels can cause it to take over or delete Pods you created separately.

To change the replica count or image, edit the Deployment.

_Usage:_

"I scaled the ReplicaSet to five, and it went straight back to three."

"The Deployment owns it and still says three. Change the replica count on the Deployment."

### Kubernetes Service

A [Kubernetes](#kubernetes) Service is an object that gives a changing set of [Pods](#pod) one stable address. Pods are replaced often and get new IPs each time, so clients connect to the Service's virtual IP or DNS name instead. A Service named `api` in namespace `shop` resolves at `api.shop.svc.cluster.local`, or `api` from within the same namespace.

A label selector chooses the Pods. Matching Pods are recorded in EndpointSlice objects, and kube-proxy, a component on each node, usually routes new connections to those passing their [readiness probe](#readiness-probe).

| Type           | Reachable from                  | Typical use                                |
| -------------- | ------------------------------- | ------------------------------------------ |
| `ClusterIP`    | Inside the cluster (default)    | Service-to-service traffic                 |
| `NodePort`     | A port on every node            | Simple external access, testing            |
| `LoadBalancer` | A cloud load balancer           | Exposing one service publicly              |
| `ExternalName` | DNS alias to an outside host    | Pointing at an external system             |

A "headless" Service, with `clusterIP: None`, has no virtual IP. DNS returns the individual Pod addresses, which a [StatefulSet](#statefulset) uses to give each replica a stable name.

A frequent failure is a Service with no ready endpoints. Connections fail even though Pods are running, usually because the selector does not match the Pod labels or no Pod is ready. Check with `kubectl get endpointslices -l kubernetes.io/service-name=api`. A wrong `targetPort` gives a similar symptom with endpoints present.

Balancing happens per connection, not per request. A client using one long-lived HTTP/2 or gRPC connection sends all its requests to one Pod, so one replica can be overloaded while others sit idle. Client-side balancing or a [service mesh](#service-mesh) can spread those requests. For HTTP routing from outside the cluster, use [Ingress](#ingress).

_Usage:_

"The pods are healthy but the Service just times out."

"Look at its endpoints. If the list is empty, the selector probably says `app: api` while the pods are labelled `app: api-server`."

### Ingress

An Ingress is a [Kubernetes](#kubernetes) object that defines how HTTP and HTTPS traffic from outside the cluster reaches internal [Services](#kubernetes-service), based on hostname and URL path. One public entry point can route `app.example.com/api` to one Service and `app.example.com/` to another, and can terminate TLS (decrypt HTTPS) using a certificate stored in a [Kubernetes Secret](#kubernetes-secret).

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web
spec:
  ingressClassName: traefik
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api
                port: { number: 80 }
```

An Ingress object is only a set of rules, and Kubernetes itself does not act on them. An ingress controller, which is a reverse proxy running in the cluster such as Traefik or HAProxy, watches these objects and configures itself. This explains a common first experience: the Ingress is created without errors, its address stays blank, and nothing responds, because no controller is installed or `ingressClassName` does not match one.

Errors at this layer usually point to a specific cause:

| Symptom        | Likely cause                                                |
| -------------- | ----------------------------------------------------------- |
| 404            | No rule matches the host or path                            |
| 502            | Controller reached a backend that refused or dropped it     |
| 503            | The Service has no ready [Pods](#pod)                   |
| 504            | Backend too slow for the proxy's timeout                    |
| Wrong certificate | TLS host or Secret mismatch, or Secret in another namespace |

Exact responses vary by controller. Behaviour beyond basic routing, such as rewrites, rate limits, and timeouts, is usually set through controller-specific annotations, so an Ingress written for one controller may not work the same on another.

The Ingress API is frozen, and new features go to its successor, the Gateway API, which separates infrastructure and routing concerns into distinct objects. Ingress remains supported and widely used.

_Usage:_

"I created the Ingress, but the address column is empty and the domain doesn't resolve."

"Is there a controller in this cluster? The object does nothing until something implements it, and your `ingressClassName` has to match."

### ConfigMap

A ConfigMap is a [Kubernetes](#kubernetes) object that stores non-confidential configuration as key-value pairs, separately from the [container image](#container-image). One image can then run in staging and production with different settings, which supports promoting a single [build artifact](#build-artifact). A ConfigMap is limited to 1 MiB and must be in the same namespace as the [Pods](#pod) using it. Credentials belong in a [Kubernetes Secret](#kubernetes-secret) or external [secrets management](#secrets-management).

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-config
data:
  LOG_LEVEL: info
  FEATURE_URL: https://flags.internal
```

The way a Pod consumes a ConfigMap decides what happens when it changes:

| Consumed as              | After the ConfigMap is edited                            |
| ------------------------ | -------------------------------------------------------- |
| Environment variable     | No change until the [container](#container) restarts |
| Mounted volume           | Files update after a delay, and the app must re-read them |
| Volume with `subPath`    | No change until the container restarts                   |

The familiar symptom is editing a value, seeing the new value in `kubectl get configmap`, and finding that the application still behaves the old way. Editing a ConfigMap does not restart anything and does not trigger a [Kubernetes Deployment](#kubernetes-deployment) rollout. Run `kubectl rollout restart deployment/api`, or include a hash of the config as an annotation in the Pod template so that a config change produces a new rollout. [Helm](#helm) charts commonly do the second.

A shared, mutable ConfigMap also weakens [rollback](#rollback), because rolling back the Deployment does not restore old config values. Versioned names such as `api-config-v12`, referenced from the Pod template, tie configuration to the release.

A missing ConfigMap blocks startup. A missing environment reference shows `CreateContainerConfigError`, and a missing volume leaves the Pod in `ContainerCreating` with a mount error in its events.

_Usage:_

"I changed LOG_LEVEL to debug an hour ago and still see nothing."

"It's injected as an env var from the ConfigMap, so running containers keep the old value. Do a rollout restart."

### Kubernetes Secret

A [Kubernetes](#kubernetes) Secret is an object for holding small pieces of sensitive data, such as passwords, API tokens, and TLS certificates, so they stay out of [container images](#container-image) and [Pod](#pod) specs. Pods consume Secrets as environment variables or mounted files. The size limit is 1 MiB.

Secret values are base64-encoded, and base64 is a reversible text format that provides no protection. Anyone who can read the object can decode it with one command:

```sh
echo 'cGFzc3dvcmQ=' | base64 --decode
```

By default the API server also stores Secrets unencrypted in etcd, the cluster's database. A frequent mistake is committing a Secret manifest to Git because the values look scrambled. Rotate any value exposed this way.

Protecting Secrets takes several separate measures:

| Measure                | What it protects against                              |
| ---------------------- | ----------------------------------------------------- |
| Encryption at rest     | Someone reading etcd or its backups                   |
| RBAC (role-based access control) | Users and workloads reading Secrets via the API |
| Namespace separation   | Workloads in other namespaces mounting them           |
| External secret store  | Long-lived values living only in the cluster          |

Access rules have a gap that is easy to miss. Anyone allowed to create a Pod in a namespace can mount any Secret in that namespace and read it, even without permission to `get` Secrets directly. For stronger control, use dedicated [secrets management](#secrets-management) and sync or inject values at runtime.

Encoding also causes a login failure. `echo "hunter2" | base64` includes a trailing newline, so the application sends a password ending in `\n` and authentication fails. Use `echo -n`, `kubectl create secret`, or the `stringData` field, which accepts plain text.

Updates behave like those of a [ConfigMap](#configmap): environment variables keep the old value until the [container](#container) restarts.

_Usage:_

"It's fine to commit this Secret YAML, the password is encoded."

"That's base64, so anyone can decode it. Kubernetes doesn't encrypt it for you. Rotate that password and keep it in Vault."

### StatefulSet

A StatefulSet is a [Kubernetes](#kubernetes) workload object for applications whose replicas are not interchangeable, such as databases, message brokers, and consensus systems. Each [Pod](#pod) gets a stable ordinal name (`db-0`, `db-1`, `db-2`) and its own persistent volume. If `db-1` is replaced, the new Pod is also `db-1` and reattaches the same storage.

| Aspect       | [Deployment](#kubernetes-deployment) | StatefulSet                   |
| ------------ | ------------------------------------------ | ----------------------------- |
| Pod names    | Random suffix                              | Stable ordinal                |
| Storage      | Usually shared or none                     | One volume claim per Pod      |
| Start order  | Parallel                                   | In order, 0 to N-1 by default |
| Update order | Surge and replace                          | Reverse order, N-1 to 0       |
| Network      | Shared [Service](#kubernetes-service) address | Per-Pod DNS name via a headless Service |

A headless Service (one with no virtual IP) gives each replica an address such as `db-0.db.prod.svc.cluster.local`, so peers and clients can reach a specific member. Storage is declared in `volumeClaimTemplates`, which create one PersistentVolumeClaim (a request for durable storage) per Pod.

A StatefulSet provides identity and ordering only. Replication, leader election, and failover belong to the software itself or to a [Kubernetes Operator](#kubernetes-operator).

Three behaviours often cause problems. Scaling down or deleting a StatefulSet leaves its volumes behind by default, and a later scale-up reattaches the old data. When a node becomes unreachable, its Pod is not recreated elsewhere, because two Pods with one identity could corrupt a shared volume. It stays stuck until the node returns or someone confirms the node is gone. With default ordering, a Pod that never becomes ready blocks the rollout, and it can stay blocked after the template is fixed until the broken Pod is deleted.

_Usage:_

"`db-2` has been Terminating for twenty minutes since that node died. Why isn't it rescheduled?"

"A StatefulSet won't start a second `db-2` until it knows the first is gone. Confirm the node is really dead before forcing it."

### DaemonSet

A DaemonSet is a [Kubernetes](#kubernetes) workload object that runs one copy of a [Pod](#pod) on every eligible node. When a node joins the cluster it receives the Pod automatically, and when a node is removed its copy goes with it. There is no replica count to set, because the number of nodes determines the number of Pods.

DaemonSets suit software tied to the machine instead of to an application:

| Use                | Example job                                        |
| ------------------ | -------------------------------------------------- |
| Log collection     | Reads [container](#container) log files from the node's disk     |
| Metrics            | Reports node CPU, memory, and disk                 |
| Networking         | Runs the cluster network plugin or kube-proxy      |
| Storage            | Runs a node driver that mounts volumes             |
| Security           | Watches system calls or file changes on the host   |

Such Pods often use `hostPath` volumes, which mount a directory from the node, or host networking, so a compromised agent can affect the whole node. Grant only the access it needs.

The usual complaint is an agent missing from some nodes, which leaves gaps in logs or metrics from those machines. The cause is often a taint, a marker on a node that repels Pods unless they declare a matching toleration. Dedicated GPU nodes and [control plane](#control-plane) nodes are commonly tainted. Compare `DESIRED` with the node count in `kubectl get daemonset`, then add the required toleration. A `nodeSelector` can also limit a DaemonSet deliberately to a subset of nodes.

A DaemonSet's cost multiplies by node count, because its [resource requests](#resource-requests-and-limits) are reserved on every machine.

Updates use a rolling strategy by default, replacing one node's Pod at a time (`maxUnavailable: 1`). A bad version of a networking or storage agent can degrade every node it reaches, so watch the first few and run `kubectl rollout undo` at the first failure.

_Usage:_

"We have no logs from the GPU nodes."

"Those nodes are tainted, so the logging DaemonSet never scheduled there. Add a toleration for the GPU taint."

### Liveness probe

A liveness probe is a periodic health check, run by the kubelet (the agent on each node), that decides whether a [container](#container) should be restarted. When the probe fails `failureThreshold` times in a row, 3 by default, the kubelet kills the container and the [Pod](#pod)'s restart policy applies. The Pod stays on the same node and keeps its IP address.

```yaml
livenessProbe:
  httpGet:
    path: /livez
    port: 8080
  periodSeconds: 10
  timeoutSeconds: 2
  failureThreshold: 3
```

A probe can be an HTTP request (a status from 200 to 399 passes), a TCP connection, a gRPC health check, or a command that must exit with 0.

Liveness probes exist for failures that a restart can fix, such as a deadlock where the process is still running but no longer doing work. Without one, [Kubernetes](#kubernetes) only restarts a container when its process exits.

A badly designed probe can cause an outage. A frequent mistake is having the liveness endpoint check the database. When the database slows down, every replica fails its probe and restarts together, removing capacity and adding a burst of reconnections. A restart cannot fix a dependency, so the liveness check should only cover the process itself. Dependency checks belong in a [readiness probe](#readiness-probe), which takes a Pod out of traffic without restarting it.

Timing causes the other common failure. An application that needs a minute to start gets killed before it finishes, restarts, and is killed again, which shows up as [CrashLoopBackOff](#crashloopbackoff). Add a startup probe, which holds off liveness and readiness checks until it first succeeds. The default `timeoutSeconds` is 1, so a busy but healthy process can miss probes under load and be restarted.

To confirm the cause, look for `Liveness probe failed` in the events from `kubectl describe pod`.

_Usage:_

"Every pod restarted at once when Postgres had that slow spell."

"Our liveness probe queries the database. Move that check to readiness, and have liveness confirm only that the process responds."

### Readiness probe

A readiness probe is a periodic check, run by the kubelet (the agent on each node), that decides whether a [Pod](#pod) should receive traffic. While it fails, the Pod is marked not ready and removed from the endpoints of any [Service](#kubernetes-service) that selects it. The [container](#container) keeps running and is not restarted. When the probe passes again, the Pod returns to rotation.

[Kubernetes](#kubernetes) has three probe types, which answer different questions:

| Probe                                 | Question                  | On failure                        | Runs                  |
| ------------------------------------- | ------------------------- | --------------------------------- | --------------------- |
| Startup                               | Has it finished starting? | Container restarted at threshold  | Until first success   |
| [Liveness](#liveness-probe)     | Is the process working?   | Container restarted               | After startup         |
| Readiness                             | Can it take traffic now?  | Removed from Service endpoints    | After startup, continuously |

Without a readiness probe, a Pod counts as ready once its containers start. Traffic then reaches an application that is still loading data or opening connections, and users see a burst of errors on every deploy.

Readiness also controls [rolling deployments](#rolling-deployment). A [Deployment](#kubernetes-deployment) only continues replacing old Pods as new ones become ready, so a version that never passes stalls the rollout while the old version keeps serving. That protection disappears if the probe always returns success.

Shared dependencies need caution. If every replica's readiness depends on one cache and that cache goes down, all Pods leave the endpoints at once, and a partial degradation becomes a full outage. Consider serving reduced functionality instead.

Shutdown is a separate gap. When a Pod is deleted, endpoint removal and the `SIGTERM` signal happen at about the same time, so a few requests can still arrive after the process begins to exit. Keep serving briefly after `SIGTERM`, or add a short `preStop` delay.

_Usage:_

"We get a spike of 503s for about thirty seconds on every deploy."

"The new pods take traffic before the cache warms up. Add a readiness probe that passes only once it's loaded."

### Resource requests and limits

Resource requests and limits are per-[container](#container) CPU and memory settings in [Kubernetes](#kubernetes). A request is the amount the scheduler reserves when choosing a node. A limit is the ceiling enforced at runtime through [cgroups](#cgroup). CPU is measured in cores or millicores (`500m` is half a CPU), and memory in bytes (`256Mi`).

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    memory: 512Mi
```

| Aspect             | Request                                     | Limit                       |
| ------------------ | ------------------------------------------- | --------------------------- |
| Used by            | Scheduler, and the kernel for CPU shares    | Kernel at runtime           |
| Exceeding CPU      | Allowed when the node has spare CPU         | Throttled (paused)          |
| Exceeding memory   | Allowed, but raises eviction risk           | Container is [OOMKilled](#oomkilled) |

Scheduling uses requests, not actual usage. A [Pod](#pod) stays Pending with an `Insufficient cpu` or `Insufficient memory` event when no node has enough unreserved capacity, even if the machines are mostly idle. Requests set too low let the scheduler pack too many Pods on one node.

If only a limit is set, the request defaults to the same value. The settings also determine a Pod's quality of service class:

| Class      | Condition                                          |
| ---------- | -------------------------------------------------- |
| Guaranteed | Every container has CPU and memory requests equal to limits |
| Burstable  | At least one request or limit, but not Guaranteed  |
| BestEffort | No requests or limits at all                       |

When a node runs short of memory, the kubelet evicts Pods, generally starting with those using more than their request. BestEffort Pods have no request, so they are exposed first.

A common starting point is to set memory requests from observed usage, with a limit at or slightly above that value. CPU limits can throttle a service while the node has idle CPU, so some teams set only a CPU request. The [Horizontal Pod Autoscaler](#horizontal-pod-autoscaler) calculates CPU utilisation against the request.

_Usage:_

"The pod's stuck in Pending, but the nodes are only at 20% CPU."

"Scheduling looks at requests, not usage. Those nodes are fully reserved, and this pod is asking for four cores. Lower its resource requests."

### OOMKilled

OOMKilled is the status [Kubernetes](#kubernetes) reports when the Linux kernel's out-of-memory (OOM) killer ends a [container](#container). It usually means the container reached the memory limit in its [resource requests and limits](#resource-requests-and-limits), which is enforced by a [cgroup](#cgroup). It can also happen when the whole node runs out of memory.

The kill uses `SIGKILL`, which a process cannot catch, so the application cannot log an error or shut down cleanly. The typical symptom is a service that restarts at random with logs that stop without explanation. The evidence is in the [Pod](#pod) status:

```text
Last State:  Terminated
  Reason:    OOMKilled
  Exit Code: 137
```

Exit code 137 means 128 plus signal 9 (`SIGKILL`). It does not prove an OOM kill on its own, because any `SIGKILL` produces it, including one sent when a process ignores a shutdown request. Check for the `OOMKilled` reason as well. Repeated kills usually lead to [CrashLoopBackOff](#crashloopbackoff).

| Memory graph before the kill | Likely cause                  | What to do                              |
| ---------------------------- | ----------------------------- | --------------------------------------- |
| Climbs steadily over hours   | Memory leak                   | Take a heap profile and fix the leak    |
| Sudden spike                 | Large request, file, or batch | Stream data or bound the work size      |
| Flat, close to the limit     | Limit too low                 | Raise it based on observed usage        |
| Container low, node full     | Node memory pressure          | Fix requests, or add node capacity      |

A heap setting such as the JVM's `-Xmx` or Node's `--max-old-space-size` covers only part of a process's memory. Thread stacks, native buffers, and non-reclaimable page cache also count toward the limit, so leave headroom.

Raising the limit is the right fix only when usage is legitimately higher. With a leak, it just delays the next kill.

_Usage:_

"The worker dies every few hours and the logs just stop."

"Check the last state. If it's OOMKilled with exit code 137, the kernel killed it, so it never had a chance to log anything."

### CrashLoopBackOff

CrashLoopBackOff is a [Kubernetes](#kubernetes) status meaning a [container](#container) keeps exiting and the kubelet (the agent on each node) is waiting before restarting it again. It describes the restart pattern. The cause is whatever made the process exit.

The wait doubles after each failed restart: 10 seconds, 20, 40, and so on, capped at five minutes. The timer resets once a container has run for 10 minutes without a problem. This is why a [Pod](#pod) can look as though nothing is happening for several minutes, then briefly show Running, then return to CrashLoopBackOff.

The first diagnostic steps are:

```sh
kubectl logs my-pod --previous
kubectl describe pod my-pod
```

The `--previous` flag matters because the current container may have just started and logged nothing, while the crashed instance holds the error. `describe` shows the last exit code and recent events.

| Exit code | Meaning                 | Usual cause                                                  |
| --------- | ----------------------- | ------------------------------------------------------------ |
| 0         | Process finished        | A one-off script run as a long-lived service                 |
| 1         | Application error       | Bad config, missing env var, failed connection at startup    |
| 126       | Cannot execute          | A script without execute permission                          |
| 127       | Command not found       | A shell entrypoint calling a missing program                 |
| 137       | `SIGKILL`               | [OOMKilled](#oomkilled), or a forced stop after a failed probe |
| 143       | `SIGTERM`               | Asked to stop, often by a failed [liveness probe](#liveness-probe) |

Exit code 0 can also produce this status. A [Deployment](#kubernetes-deployment) requires `restartPolicy: Always`, so a container that completes successfully is restarted. Use a Job (a workload that runs to completion) for one-off tasks.

If logs are empty, the runtime may have failed to start the command, which `describe` reports as a start error. Check the entrypoint and [image](#container-image) architecture, or debug with an ephemeral container through `kubectl debug`.

Restarting or deleting the Pod rarely helps, since the replacement hits the same failure.

_Usage:_

"The new release is in CrashLoopBackOff and the logs are empty."

"Run logs with `--previous`, you're looking at the fresh container. And check the exit code in describe."

### Horizontal Pod Autoscaler

A Horizontal [Pod](#pod) Autoscaler (HPA) is a [Kubernetes](#kubernetes) controller that adjusts the number of Pod replicas in a workload, such as a [Deployment](#kubernetes-deployment), according to observed metrics. "Horizontal" means adding or removing replicas, as opposed to giving each one more CPU or memory. By default it re-evaluates every 15 seconds using this calculation:

```text
desired = ceil(current replicas × current metric / target metric)
```

With 4 replicas averaging 90% CPU against a 60% target, the result is `ceil(4 × 90 / 60) = 6`. The controller skips changes when the ratio of current to target is within 10% of 1.0, which prevents constant small adjustments.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api
spec:
  scaleTargetRef: { apiVersion: apps/v1, kind: Deployment, name: api }
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target: { type: Utilization, averageUtilization: 60 }
```

CPU utilisation is calculated against the Pod's CPU request from its [resource requests and limits](#resource-requests-and-limits). If `kubectl get hpa` shows `<unknown>` as the current value and nothing scales, the usual cause is a missing CPU request or a missing metrics-server (the add-on that reports resource usage).

Other common problems:

| Symptom                              | Cause                                         | Fix                                    |
| ------------------------------------ | --------------------------------------------- | -------------------------------------- |
| Replicas reset on each deploy        | Manifest also sets `spec.replicas`            | Remove `replicas` from the manifest    |
| Slow to scale down                   | Default five-minute stabilisation window      | Tune `behavior` if needed              |
| New Pods stay Pending                | Cluster has no spare node capacity            | Add a node autoscaler                  |
| No scaling during overload           | Bottleneck is I/O, so CPU stays low           | Scale on queue depth or request rate   |

The HPA reacts after load has already risen, and new Pods still need time to pull [images](#container-image) and pass their [readiness probe](#readiness-probe). For sharp spikes, keep `minReplicas` high enough to absorb the first wave.

_Usage:_

"Traffic tripled and the HPA didn't add a single pod."

"Its target says `<unknown>`. The [container](#container) has no CPU request, so it has nothing to compute utilisation against."

### Helm

Helm is a package manager for [Kubernetes](#kubernetes). It bundles the manifests an application needs into a versioned package called a chart, fills in templates using configuration values, applies the result to a cluster, and records each install or upgrade so it can be reversed.

| Term       | Meaning                                                     |
| ---------- | ----------------------------------------------------------- |
| Chart      | A package of templates, defaults, and metadata              |
| Values     | Configuration supplied to the templates                     |
| Template   | A manifest with placeholders, written in Go template syntax |
| Release    | One installed instance of a chart in a cluster              |
| Revision   | A numbered version of a release                             |
| Repository | A place that hosts charts, often an OCI [container registry](#container-registry) |

A template fragment looks like this:

```yaml
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: api
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

The same chart can then deploy to staging and production with a different values file. Charts carry a [semantic version](#semantic-versioning), so a team can install third-party software at a known version.

Generating YAML through text templates has known failure modes. Wrong indentation produces invalid manifests, or valid ones with a field in the wrong place. A misspelled key in a values file is usually ignored without any warning, so an override appears to do nothing. Run `helm template` or `helm upgrade --dry-run` to inspect the rendered output before applying it.

A successful `helm upgrade` can also mislead. Without a wait option, Helm reports success once the API server accepts the resources, while the new [Pods](#pod) may be crashing. Use `--wait` with a timeout in [pipelines](#pipeline), and `--rollback-on-failure` to undo a failed upgrade. Helm 3 calls this flag `--atomic`, and Helm 4 renamed it.

`helm rollback` restores an earlier revision's manifests by creating a new revision. It does not undo data changes. Helm stores release history in [Secrets](#kubernetes-secret) in the release's namespace by default, so deleting those Secrets makes Helm lose track of the release.

_Usage:_

"I set `replicas: 5` in the values file, and it's still running two."

"The chart reads `replicaCount`, so Helm ignored your key. Run `helm template` and check the output."

### Kubernetes Operator

A [Kubernetes](#kubernetes) Operator is a custom controller that automates the operation of a specific application by extending the Kubernetes API. It encodes tasks a human administrator would otherwise perform, such as setting up replication, taking backups, handling failover, and running version upgrades.

An Operator has three parts:

| Part                               | Role                                                  |
| ---------------------------------- | ----------------------------------------------------- |
| CustomResourceDefinition (CRD)     | Adds a new object type to the Kubernetes API          |
| Custom resource                    | An instance of that type, stating what you want       |
| Controller                         | Watches those objects and acts to make them true      |

With a PostgreSQL Operator such as CloudNativePG, a database cluster becomes one short manifest:

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: orders-db
spec:
  instances: 3
  storage:
    size: 20Gi
```

The controller creates the [Pods](#pod), storage, and [Services](#kubernetes-service), configures replication, and promotes a replica if the primary fails. It runs a reconcile loop, the same pattern built-in controllers use: read the desired state, inspect the actual state, and act on the difference.

Operators are useful for stateful software, where a [StatefulSet](#statefulset) supplies identity and storage but knows nothing about the application's failover process. For a stateless app, a [Deployment](#kubernetes-deployment) or [Helm](#helm) chart is usually enough.

An Operator is extra software you must run and trust. It often holds broad cluster permissions, its CRDs need upgrading alongside it, and a bug in it acts automatically on production systems. If it stops running, nothing reconciles, so failover and repair stop too.

A recognisable failure comes from removing an Operator before its resources. Custom resources often carry a finalizer, a marker that blocks deletion until a controller finishes cleanup. With the controller gone, nothing clears the marker, and `kubectl delete` hangs with the object stuck in `Terminating`. Delete custom resources first, then uninstall the Operator. Removing a finalizer by hand skips that cleanup and can leave external resources behind.

_Usage:_

"This custom resource has been stuck in Terminating since yesterday."

"Its finalizer is waiting for the Kubernetes Operator, and someone uninstalled it first. Reinstall it so that it can finish the cleanup."

### Service mesh

A service mesh is an infrastructure layer that handles network communication between services using proxies, so applications do not each implement that logic. Traffic passes through a proxy on the way in and out, and a central component configures all the proxies. Istio and Linkerd are examples.

| Feature         | What the mesh does                                           |
| --------------- | ------------------------------------------------------------ |
| Mutual TLS      | Encrypts traffic and verifies both sides' identities         |
| Traffic routing | Splits by weight, for example for a [canary release](#canary-release) |
| Reliability     | Applies retries, timeouts, and circuit breaking (cutting off a failing dependency) |
| Telemetry       | Reports request rate, errors, and latency per service        |

A mesh has a data plane, the proxies that carry traffic, and its own control component, separate from the [Kubernetes](#kubernetes) [control plane](#control-plane). The traditional data plane injects a sidecar proxy [container](#container) into each [Pod](#pod). Newer designs, such as Istio's ambient mode, move much of the work to shared per-node proxies.

A mesh adds a hop to every call, proxy memory and CPU, and another system to upgrade and debug. Adopt one for a specific need, such as required encryption between many services. A few services can often manage with a library and a plain [Service](#kubernetes-service).

Sidecars have known startup and shutdown problems. An application that makes a network call before its proxy is ready fails at boot, which can look random. A Job (a workload that runs to completion) can hang because its main container finished while the sidecar kept running. Kubernetes native sidecars, declared as init containers with `restartPolicy: Always`, address both.

Retries also need coordination. If three layers each try a request three times, one failing call can become 27 attempts against a service that is already struggling.

_Usage:_

"The app fails its first DB call on boot, but only since we added the service mesh."

"The sidecar isn't ready when your app starts. Switch to native sidecars so the proxy comes up first."

## Section 4 — Infrastructure as Code

### Infrastructure as code

Infrastructure as code (IaC) is the practice of defining servers, networks, databases, and permissions in version-controlled files, then using a tool to create and update them. The files replace manual work in a cloud console. A change to infrastructure becomes a commit that can be reviewed, tested, and reverted.

It addresses a familiar situation: production was built by hand over several years, nobody knows why one firewall rule exists, and staging differs in ways no one can list. When someone has to rebuild after a failure, the only documentation is the running system. With IaC, the definition is written down and can be run again.

IaC tools take one of two approaches:

| Approach    | You write                  | Tool's job                              | Examples                        |
| ----------- | -------------------------- | --------------------------------------- | ------------------------------- |
| Declarative | The desired end state      | Work out the changes needed             | [Terraform](#terraform), CloudFormation |
| Imperative  | The steps to perform       | Run those steps in order                | Shell scripts, cloud SDK calls  |

Declarative tools are the usual choice because the same file can be applied repeatedly. If the infrastructure already matches, nothing happens. An imperative script that creates a server will often create a second one when it runs again, unless it is written to check first.

A typical workflow puts changes through a [pipeline](#pipeline): open a pull request, generate a preview of what will change, have someone review that preview, and apply after merging. The preview matters, because a small edit can have a large effect, such as replacing a database. [Policy as code](#policy-as-code) can check the preview automatically.

IaC only works while the code stays authoritative. Emergency edits in the console create [configuration drift](#configuration-drift), and the next apply may silently undo them. Credentials should also stay out of these files and come from [secrets management](#secrets-management).

_Usage:_

"How long would it take to recreate the staging VPC if someone deleted it?"

"Days, since it was all clicked together. We should bring it under infrastructure as code before we need to find out."

### Terraform

Terraform is a declarative [infrastructure as code](#infrastructure-as-code) tool from HashiCorp. You describe resources in HCL (HashiCorp Configuration Language), and Terraform works out what to create, change, or delete to reach that description. Plugins called providers connect it to AWS, Google Cloud, Azure, Cloudflare, and many other APIs. OpenTofu is an open-source fork with a largely compatible workflow.

```hcl
resource "aws_s3_bucket" "logs" {
  bucket = "acme-app-logs"

  lifecycle {
    prevent_destroy = true
  }
}
```

| Command             | What it does                                                  |
| ------------------- | ------------------------------------------------------------- |
| `terraform init`    | Downloads providers and connects to the state backend         |
| `terraform plan`    | Shows the changes it would make, without making them          |
| `terraform apply`   | Makes the changes after confirmation                          |
| `terraform destroy` | Deletes everything the configuration manages                  |

To build a plan, Terraform compares three things: your configuration, its record of what it manages in [Terraform state](#terraform-state), and the real resources as reported by the provider. Differences found in the real resources are [configuration drift](#configuration-drift).

A plan can propose destroying a database after a small edit. Some attributes cannot be changed on an existing resource, so changing one means deleting and recreating it. The plan marks this as `-/+` with `# forces replacement` beside the attribute. Renaming a resource block has a similar effect, because Terraform sees one resource removed and another added. A `moved` block tells it that the two are the same.

Read every plan before applying, especially lines containing `destroy` or `replace`. Apply the saved plan with `terraform plan -out=tfplan` followed by `terraform apply tfplan`, so that what was reviewed is what runs. Use `prevent_destroy` on resources that hold data. It makes any plan that would destroy the resource fail, although removing the block from the configuration removes that protection.

_Usage:_

"Terraform wants to replace the production database. I only changed one setting."

"That setting forces replacement. Stop before applying, and find a way to migrate it without recreating the instance."

### Terraform state

[Terraform](#terraform) state is the data file in which Terraform records the real resources it manages. It maps each block in your configuration, such as `aws_instance.web`, to an actual ID in the provider, such as `i-0abc123`, along with the attributes last seen. Without this mapping, Terraform has no way to know that a server in your account belongs to a given block.

By default state is a local file called `terraform.tfstate`. Teams store it in a remote backend so that everyone works from the same copy:

```hcl
terraform {
  backend "s3" {
    bucket       = "acme-terraform-state"
    key          = "prod/network.tfstate"
    region       = "eu-west-1"
    encrypt      = true
    use_lockfile = true
  }
}
```

Most state incidents fall into a few patterns:

| Problem                  | What you see                                    | Response                                   |
| ------------------------ | ----------------------------------------------- | ------------------------------------------ |
| State lost               | Plan wants to create everything again           | Restore a backup, or import resources      |
| Concurrent applies       | State lock error, or corruption if no locking   | Use locking, and run applies in a [pipeline](#pipeline) |
| Stale lock               | `Error acquiring the state lock` after a crash  | Confirm nothing is running, then `force-unlock` |
| Resource renamed         | Plan shows destroy and create                   | Add a `moved` block                        |
| Resource created by hand | Terraform ignores it                            | Bring it in with an `import` block         |

State often holds secrets. Database passwords and generated keys are stored in it in plain text, even when marked `sensitive` in the configuration, which only hides them from command output. Never commit it to Git. Keep it in an encrypted backend with restricted access, and treat read access to state as access to those credentials.

Enable versioning on the storage bucket, so that a damaged state file can be restored. Avoid hand-editing the file, and use `terraform state` subcommands when you need to change it. Splitting large projects into several smaller states also reduces the impact of a mistake and keeps plans fast.

_Usage:_

"Terraform says it needs to create all 140 resources, but they already exist."

"It's reading empty Terraform state. Check that the backend key is right before anyone applies anything."

### Configuration drift

Configuration drift is the difference that develops between what your configuration declares and what is actually running. It usually starts with a reasonable action: during an incident, someone opens a port in the cloud console, edits a [Kubernetes Deployment](#kubernetes-deployment) with `kubectl edit`, or changes a setting on one server over SSH. The fix works, and nobody updates the code.

Drift has two recognisable consequences. The first is a hotfix that disappears. Weeks later, someone runs a routine [Terraform](#terraform) apply, the tool restores the declared value, and the incident returns. The second is environments that behave differently for reasons no one can find, where a bug appears in production and cannot be reproduced in staging because their settings are no longer alike.

Detection depends on the layer:

| Layer          | How to detect drift                                                     |
| -------------- | ----------------------------------------------------------------------- |
| Cloud resources | A scheduled `terraform plan`, alerting on any non-empty result         |
| [Kubernetes](#kubernetes) | A [GitOps](#gitops) controller reporting an out-of-sync state       |
| Servers        | A [configuration management](#configuration-management) run in check mode |

Once found, each difference needs a decision:

| Decision | When                                     | Action                                  |
| -------- | ---------------------------------------- | --------------------------------------- |
| Adopt    | The live change is correct               | Update the code to match                |
| Revert   | The live change was a mistake            | Apply the code                          |
| Ignore   | Something else legitimately manages it   | Exclude the field, such as with `ignore_changes` |

Some differences are expected. A [Horizontal Pod Autoscaler](#horizontal-pod-autoscaler) changes replica counts by design, so the replica field should not be owned by the manifest as well.

To prevent drift, restrict write access to production, keep a fast emergency route through the normal [pipeline](#pipeline), and require a follow-up commit after any manual fix. [Immutable infrastructure](#immutable-infrastructure) reduces drift on servers because they are replaced instead of edited.

_Usage:_

"The rate limit we raised during Friday's incident is back to the old value."

"That was a console change, so it was configuration drift. Today's apply reverted it. Put the new value in the Terraform code."

### Immutable infrastructure

Immutable infrastructure is an approach in which running servers and [containers](#container) are never modified after they are deployed. To change anything, whether an application version, a package, or a setting, you build a new image and replace the old instances with it. The image can be a [container image](#container-image) or a virtual machine image built with a tool such as Packer.

| Aspect          | Mutable                           | Immutable                         |
| --------------- | --------------------------------- | --------------------------------- |
| Applying a change | Update the running server       | Build a new image and replace it  |
| Server history  | Accumulates over its lifetime     | Exactly what the image contains   |
| Failed update   | Can leave a partial state         | New instance fails, old one remains |
| Going back      | Reverse each change               | Redeploy the previous image       |

The approach addresses the "snowflake server", a machine that has been patched and adjusted by hand for years until nobody can recreate it, and everyone avoids rebooting it. Replacing instances routinely prevents that history from forming and removes most [configuration drift](#configuration-drift) on the host.

Immutability has requirements. Anything that must survive replacement, such as uploaded files, databases, and logs, has to live outside the instance in external storage or a logging service. Build and deployment must be automated, because a small change now means a full build and rollout. Configuration that varies by environment is supplied at startup, so that one [build artifact](#build-artifact) works everywhere.

Debugging habits also change. A fix patched onto a live server vanishes at the next deployment or scaling event, so it has to go into the image definition. A [rollback](#rollback) means redeploying the previous image, provided data changes remain compatible. [Kubernetes](#kubernetes) works this way by default: changing a [Pod](#pod) template creates new Pods and removes the old ones.

_Usage:_

"I patched the config on that box last week. Why has the bug come back?"

"We run immutable infrastructure, so autoscaling replaced that instance. The fix needs to go in the image build."

### GitOps

GitOps is an operating model in which a Git repository holds the declared state of a system, and an automated agent continuously makes the live system match it. To change production, you merge a commit. Git supplies the review, audit history, and a way to revert, while the agent does the applying. Argo CD and Flux are common agents for [Kubernetes](#kubernetes).

The model differs from a traditional deployment [pipeline](#pipeline) in the direction of access:

| Aspect          | Push-based [CI](#continuous-integration)/CD                     | Pull-based GitOps                       |
| --------------- | ------------------------------------ | --------------------------------------- |
| Who applies     | The CI job                           | An agent inside the cluster             |
| Credentials     | CI holds cluster access              | Cluster holds read access to Git        |
| When it runs    | Once, on a pipeline trigger          | Continuously                            |
| Manual changes  | Remain until the next deploy         | Detected, and optionally reverted       |

A typical flow has CI build and publish a [container image](#container-image), then update the image tag in a separate configuration repository. The agent sees the commit and rolls out the change. A [rollback](#rollback) is usually a `git revert` of that commit.

Continuous reconciliation can revert manual changes. During an incident, someone runs `kubectl scale` or `kubectl edit`, sees the change take effect, and a minute later it is gone. With self-healing enabled, the agent treats the manual edit as [configuration drift](#configuration-drift) and restores the Git version. To make an emergency change stick, commit it, or explicitly suspend syncing for that application and resume it afterwards.

Plain secrets cannot be stored in Git, so teams commit encrypted values or references to an external store through [secrets management](#secrets-management). Fields changed by controllers, such as replica counts under a [Horizontal Pod Autoscaler](#horizontal-pod-autoscaler), must be excluded or they will fight the agent.

_Usage:_

"I scaled the API to ten replicas and it dropped back to four by itself."

"Argo reverted it. We use GitOps, so the repo wins. Open a PR changing the replica count."

### Configuration management

Configuration management is the use of tools to install software, write config files, manage users, and keep services running on existing servers in a defined way. Instead of logging in to each machine, you describe the desired setup once and apply it across a fleet. Ansible, Puppet, Chef, and Salt are established examples.

```yaml
- name: Ensure nginx is installed
  ansible.builtin.apt:
    name: nginx
    state: present

- name: Ensure nginx is running
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
```

The important property is idempotence: running the same configuration many times gives the same result as running it once. The task above installs nginx only when it is missing. Compare a shell script containing `echo "option" >> app.conf`, which appends a duplicate line on every run until the service refuses to start or behaves strangely.

Tools differ in how changes reach a machine:

| Model | How it works                                 | Examples      | Trade-off                                  |
| ----- | -------------------------------------------- | ------------- | ------------------------------------------ |
| Push  | A control machine connects, often over SSH   | Ansible       | No agent, but runs only when triggered     |
| Pull  | An agent on each server fetches its config   | Puppet, Chef  | Corrects drift regularly, but needs agents |

Configuration management is often confused with [infrastructure as code](#infrastructure-as-code) tools for provisioning. [Terraform](#terraform) typically creates the server, network, and disk, and configuration management sets up what runs inside the machine. The two are commonly used together.

These tools only manage what they are told to manage. A package installed by hand stays on the machine even after the playbook is corrected, so long-lived servers still gather [configuration drift](#configuration-drift). A run can also fail halfway through a fleet, leaving some hosts updated and others not.

[Container](#container)-based systems reduce the need for this work, because the software is baked into an [image](#container-image) and replaced under [immutable infrastructure](#immutable-infrastructure). Configuration management remains useful for virtual machines, network devices, and building base images.

_Usage:_

"Half the fleet has the new TLS settings and the other half doesn't."

"The Ansible run stopped at host 40. It's idempotent, so fix the failure and run the configuration management playbook again."

### Policy as code

Policy as code is the practice of writing organisational rules as machine-readable code, so that tools can evaluate changes against them automatically. Rules such as "no public storage buckets", "[containers](#container) must not run as root", or "[images](#container-image) must come from our registry" are versioned, reviewed, and tested in the same way as application code. Open Policy Agent (with its Rego language), Kyverno, and HashiCorp Sentinel are examples.

Written guidelines depend on someone remembering them during review. A policy engine gives the same answer every time and reports which rule failed.

The same rule can be enforced at several points:

| Enforcement point         | What is checked                         | Effect of a violation            |
| ------------------------- | --------------------------------------- | -------------------------------- |
| Developer machine or PR   | Manifests and config files              | Early feedback                   |
| [Pipeline](#pipeline) | A [Terraform](#terraform) plan or built image | Blocks the merge or deploy |
| [Kubernetes](#kubernetes) admission | Objects sent to the API server          | The API request is rejected      |
| Runtime audit             | Resources that already exist            | Reports or remediates            |

Earlier checks give faster feedback, while admission control is harder to bypass. An admission controller is a [control plane](#control-plane) plugin that can inspect or modify a request before it is stored. Policy engines usually attach as admission webhooks, so a blocked deploy appears as an error such as `admission webhook "..." denied the request`.

A new rule in blocking mode can break existing deployments. Run it in audit mode first, fix or exempt what it would reject, then enforce it. Each policy should tell the developer how to comply.

Admission webhooks also add an availability risk. If a webhook is set to `failurePolicy: Fail` and its service goes down, every matching API request is refused, which can block new [Pods](#pod), including the replacement for the webhook itself. Run it with multiple replicas, and exclude system namespaces.

_Usage:_

"My deploy just got rejected by an admission webhook."

"That's policy as code. The rule says containers can't run as root, so set `runAsNonRoot` and apply it again."

### Secrets management

Secrets management is the practice of controlling credentials such as passwords, API keys, tokens, and certificates throughout their lifecycle. It covers storage, access, delivery to applications, rotation (replacing them with new values), and auditing. HashiCorp Vault, AWS Secrets Manager, and Google Secret Manager are examples of dedicated stores.

The typical failure is a credential committed to Git. Deleting it in a later commit does not help, because it remains in the repository's history and in every clone. Once a secret has been pushed, treat it as compromised: revoke it, issue a new one, and check the access logs. Secret scanners in pre-commit hooks and [pipelines](#pipeline) can catch many of these before they land.

Approaches form a ladder:

| Level | Approach                     | Remaining weakness                          |
| ----- | ---------------------------- | ------------------------------------------- |
| 1     | Hardcoded in source          | Visible to anyone with the code             |
| 2     | `.env` files on servers      | Copied around, rarely rotated               |
| 3     | [CI](#continuous-integration) or platform secret store | Long-lived, limited audit                   |
| 4     | [Kubernetes Secret](#kubernetes-secret), encrypted at rest | Readable by anyone who can create [Pods](#pod) in the namespace |
| 5     | Central secrets manager      | Applications still hold static values       |
| 6     | Short-lived dynamic credentials | Depends on the issuer being available    |

Dynamic credentials are issued on demand and expire automatically, so a leaked value has a short useful life. With workload identity or OIDC federation (trading a signed identity token for temporary access), a CI job or Pod receives temporary cloud permissions without a stored credential.

Rotation fails when nobody knows which services use a key, so the first attempt causes an outage. Automate it, and support two valid credentials during the changeover.

Secrets also leak through less obvious channels: logs, error reports, [Terraform state](#terraform-state), and [image layers](#image-layer). Inject values at runtime, and keep them out of builds and output.

_Usage:_

"I removed the leaked AWS key in the next commit, so we're fine, right?"

"No, it's still in history. Revoke it now. Then move it into secrets management."

## Section 5 — Release Strategies

### Rolling deployment

A rolling deployment is a release strategy that replaces running instances of an application with a new version in small batches. Capacity stays mostly available throughout, and no second full environment is needed. It is the default update strategy for a [Kubernetes Deployment](#kubernetes-deployment).

[Kubernetes](#kubernetes) controls the pace with two settings. `maxSurge` is how many extra [Pods](#pod) may exist above the desired count, and `maxUnavailable` is how many may be missing. With 4 replicas, `maxSurge: 1`, and `maxUnavailable: 0`:

| Step | Old Pods | New Pods | What happens                         |
| ---- | -------- | -------- | ------------------------------------ |
| 1    | 4        | 1        | One new Pod starts                   |
| 2    | 3        | 1        | It becomes ready, one old Pod leaves |
| 3    | 3        | 2        | Next new Pod starts                  |
| 4    | 2        | 2        | It becomes ready, one old Pod leaves |
| 5    | 0        | 4        | The pattern repeats until complete   |

Two versions serve traffic at the same time during the rollout, which causes intermittent errors that appear only during deploys. A page from the new version calls an API endpoint, the call lands on an old Pod that lacks it, and the request fails until a refresh. Each release must stay compatible with the one before it, which is the purpose of [expand and contract](#expand-and-contract).

Two details prevent dropped requests. A [readiness probe](#readiness-probe) keeps traffic away from a new Pod until it can serve. Graceful shutdown lets an old Pod finish in-flight requests after `SIGTERM`, before the grace period (30 seconds by default) ends in `SIGKILL`.

A [rollback](#rollback) is another rolling update in the opposite direction, so it takes time. For an instant switch, use a [blue-green deployment](#blue-green-deployment). To measure impact on a small share of traffic first, use a [canary release](#canary-release).

_Usage:_

"We get a handful of 404s from the API on every deploy, then they stop."

"During the rolling deployment, new frontends hit old backends. Ship the endpoint first, then the frontend that calls it."

### Blue-green deployment

A blue-green deployment is a release strategy that keeps two production environments and moves traffic between them in one switch. "Blue" runs the current version and receives all traffic. "Green" gets the new version and is tested while idle. When it looks correct, a router or load balancer (a component that spreads traffic across servers) points everyone at green.

| Step | Blue            | Green           | What happens                           |
| ---- | --------------- | --------------- | -------------------------------------- |
| 1    | Live            | Idle            | Deploy the new version to green        |
| 2    | Live            | Under test      | Run smoke tests against green directly |
| 3    | Idle            | Live            | Switch traffic to green                |
| 4    | Idle, on standby | Live           | Watch metrics, keep blue running       |
| 5    | Next target     | Live            | Blue becomes the next release target   |

The main benefit is a fast [rollback](#rollback): if green misbehaves, the router points back at blue, with no rebuild or redeploy. Unlike a [rolling deployment](#rolling-deployment), users do not see a long period with two versions mixed. In [Kubernetes](#kubernetes), one simple form runs two [Deployments](#kubernetes-deployment) and changes the label selector on a [Service](#kubernetes-service).

Both environments typically share one database, which limits rollback. If green's release drops or renames a column, switching back to blue fails. Schema changes need to work with both versions, using [expand and contract](#expand-and-contract).

The switch also has side effects:

- Green starts with cold caches, so latency can jump after the cutover.
- Existing long-lived connections, such as WebSockets, may stay on blue until they are closed.
- DNS-based switching is slow and uneven, because clients cache old answers.
- Background workers in both environments may process the same jobs unless one is paused.

Running two full environments can double compute cost, so many teams create green on demand and remove it afterwards.

_Usage:_

"Green's error rate just doubled after the cutover."

"Flip the load balancer back to blue. That's why we keep it running after a blue-green deployment."

### Canary release

A canary release is a release strategy that sends a small share of real production traffic to a new version, compares its behaviour with the current version, and increases the share only if it stays healthy. If it degrades, traffic returns to the old version, so the failure affects a small group of users.

A typical progression looks like this, though the percentages and timings vary:

| Stage | Traffic to new version | Hold       | Promote if                         |
| ----- | ---------------------- | ---------- | ---------------------------------- |
| 1     | 1%                     | 10 minutes | Errors and latency match baseline  |
| 2     | 10%                    | 30 minutes | Same, plus key business metrics    |
| 3     | 50%                    | 30 minutes | Same                               |
| 4     | 100%                   | Done       | Old version removed                |

A canary catches problems that tests miss, because real traffic has data, clients, and load patterns that staging lacks. It only helps if someone defines what "healthy" means before the release. Compare the canary with a baseline running the old version at the same moment, because comparing against yesterday's numbers mixes in normal traffic changes. Tools such as Argo Rollouts and Flagger can automate promotion and [rollback](#rollback) based on these metrics.

Traffic splitting needs some care. A plain [Kubernetes Service](#kubernetes-service) spreads connections across [Pods](#pod), so a percentage can only be approximated by the replica ratio, such as one canary in ten Pods. Exact weights need an [Ingress](#ingress) controller or [service mesh](#service-mesh) that supports them.

Small canaries have blind spots. At 1% on a quiet service, there may be too few requests for a problem to show up, and slow failures such as memory leaks can outlast the hold time. Assign users consistently, so that one person does not bounce between versions. As in a [rolling deployment](#rolling-deployment), both versions must work with the same database.

_Usage:_

"Should we ship the new ranking model to everyone tonight?"

"Do a canary release at 5% first and compare error rate and checkout conversion against the baseline for an hour."

### Dark launch

A dark launch is the release of new code to production so that it runs against real traffic or data while users continue to see the existing behaviour. The new path does its work, but its output is discarded or logged for comparison. This tests performance and correctness under production conditions before anyone depends on the result.

A common form is shadow traffic. The service handles each request as usual and also sends a copy to the new implementation asynchronously. The user receives the old response, while engineers compare the two responses and watch the new system's latency and error rate. A [feature flag](#feature-flag) often controls which requests are mirrored, and a [service mesh](#service-mesh) can mirror them at the network level.

Dark launches are often confused with neighbouring strategies:

| Strategy                                | Runs in production? | Do users see the result? |
| --------------------------------------- | ------------------- | ------------------------ |
| Dark launch                             | Yes                 | No                       |
| [Canary release](#canary-release) | Yes                 | A small share of users   |
| Feature flag rollout                    | Yes                 | Chosen users             |

The technique suits risky backend replacements, such as a rewritten pricing engine or a new search index, because staging rarely reproduces real data shapes and volume.

Side effects are the main hazard. If a mirrored request sends an email, charges a card, or writes to a shared database, it happens twice, and customers see duplicates from a feature meant to be invisible. Stub out writes and external calls in the dark path, or point them at isolated storage.

Dark traffic is also real load, and extra calls to a shared dependency can slow the live path. Sample a small percentage first, send copies with strict timeouts, and keep a way to turn mirroring off. Remove the comparison code after launch.

_Usage:_

"How do we know the new pricing engine handles real orders?"

"Dark launch it. Mirror 10% of requests, discard its output, and diff its prices against the old engine for a week."

### Rollback

A rollback is the act of returning a system to a previous known-good version after a release causes problems. It is usually a quick way to restore service, because it reuses something that has already run in production.

The available method depends on how the release was made:

| Method                  | How it works                                                    | Typical speed |
| ----------------------- | --------------------------------------------------------------- | ------------- |
| [Feature flag](#feature-flag) off | Disables the code path with no deploy           | Seconds       |
| Traffic switch          | Points back at the old [blue-green](#blue-green-deployment) environment | Seconds to minutes |
| Abort a [canary](#canary-release) | Returns the traffic share to the old version    | Seconds to minutes |
| Redeploy old artifact   | Rolls out the previous [build artifact](#build-artifact)  | Minutes       |
| `kubectl rollout undo`  | Restores the previous [Deployment](#kubernetes-deployment) revision | Minutes |
| `git revert` with [GitOps](#gitops) | Reverts the commit and lets the agent sync | Minutes       |

A rollback sometimes fails to fix the incident. The old code is running again and errors continue, because the release changed something other than code. Common causes:

- A database migration dropped or rewrote a column that the old version needs.
- New code already wrote data in a format the old code cannot read.
- A [ConfigMap](#configmap), [Secret](#kubernetes-secret), or infrastructure change is not versioned with the release.
- The real cause was never the release, such as a failing dependency or a traffic spike.
- The previous artifact was deleted by a retention policy.

Plan for this before release: keep schema changes backward compatible using [expand and contract](#expand-and-contract), retain recent artifacts, and rehearse the procedure.

Rolling forward, meaning shipping a fix as a new version, can be safer when the fix is small and understood, or when data has changed so that old code cannot run.

_Usage:_

"We rolled back twenty minutes ago and checkout is still failing."

"Then it isn't only the code. That release ran a migration, so the rollback put old code on the new schema."

### Expand and contract

Expand and contract, also called parallel change, is a way to make a breaking change to a database schema or API through a series of compatible steps. You first add the new form alongside the old one, move everything across, and remove the old form only when nothing uses it. Old and new application versions keep working at every step.

The technique exists because deployments are not instantaneous. During a [rolling deployment](#rolling-deployment), old and new code run at the same time against one database. If a release renames `name` to `full_name` in a single migration, every old instance starts failing with "column does not exist" the moment the migration runs. Reversing the application with a [rollback](#rollback) does not help, because the old code needs a column that is gone.

Renaming that column safely takes several releases:

| Step | Phase    | Change                                         | Safe to roll back? |
| ---- | -------- | ---------------------------------------------- | ------------------ |
| 1    | Expand   | Add nullable `full_name` column                | Yes                |
| 2    | Migrate  | Deploy code that writes to both columns        | Yes                |
| 3    | Migrate  | Backfill existing rows in small batches        | Yes                |
| 4    | Migrate  | Deploy code that reads from `full_name`        | Yes, still dual-writing |
| 5    | Migrate  | Deploy code that writes only `full_name`       | To step 4. Earlier needs a re-sync |
| 6    | Contract | Drop `name` after confirming nothing uses it   | No                 |

Step 6 is the only destructive change and can wait until the team is confident. Verify the backfill first, and check for other readers such as reports and scheduled jobs.

The same pattern applies to APIs: add the new field, migrate clients, then remove the old one. [Continuous deployment](#continuous-deployment) depends on this pattern, at the cost of more releases and temporary dual-write code.

_Usage:_

"Can I rename this column in today's migration?"

"Not in one step, because the old [pods](#pod) will crash mid-deploy. Use expand and contract: add the new column first and drop the old one next week."

## Section 6 — Supply Chain

### Software supply chain

The software supply chain is everything involved in producing and delivering a piece of software: source code, third-party dependencies, base [images](#container-image), compilers, [pipelines](#pipeline), registries, and deployment tools. Your application inherits the trust of each link. Installing one package can pull in many others that nobody on the team chose directly.

A supply chain attack compromises one of those links instead of the application. The malicious code then arrives through a normal, trusted update, with no vulnerability in your own code:

| Attack              | How it works                                                        | Defence                                    |
| ------------------- | ------------------------------------------------------------------- | ------------------------------------------ |
| Typosquatting       | Publishes a package with a name close to a popular one              | Review new dependencies                    |
| Dependency confusion | Public package shadows an internal name                            | Scoped names, a controlled [artifact registry](#artifact-registry) |
| Maintainer takeover | A trusted package gets a malicious release                          | [Dependency pinning](#dependency-pinning), delayed updates |
| Install scripts     | Package runs code during installation                               | Disable scripts where possible             |
| [CI](#continuous-integration) compromise | Attacker steals pipeline secrets or alters the build                | Least privilege, pinned CI actions         |
| Tampered artifact   | Image or package is altered after the build                         | Signing and digest verification            |

Defences work in layers. Know what you ship, using an [SBOM](#sbom). Find known flaws with [image scanning](#image-scanning) and dependency audits. Control what enters the build with pinned versions and a proxy registry. Prove where artifacts came from through signing, with a tool such as Sigstore's cosign, and through provenance, which is a signed record of how and from what an artifact was built. The SLSA framework (Supply-chain Levels for Software Artifacts) defines progressive levels for build integrity.

A signature has no effect unless something checks it, so verify it at deploy time, for example with [policy as code](#policy-as-code) in a [Kubernetes](#kubernetes) admission controller.

_Usage:_

"Why does the build need network access to run a postinstall script from some transitive package?"

"It doesn't, and that's a software supply chain risk. Disable install scripts in CI and allowlist the few that need them."

### Dependency pinning

Dependency pinning is the practice of fixing each dependency to an exact version or content hash, so that a build uses the same inputs every time. Without it, a version range or moving tag can resolve to something new between two builds.

The familiar symptom is a build that passed on Friday and fails on Monday with no commits in between. A package several levels down published a new release, the range accepted it, and CI picked it up. [Semantic versioning](#semantic-versioning) ranges rely on maintainers classifying each change correctly, and on nobody publishing a malicious update.

Pinning has several levels of strength:

| Level          | Example                         | What it guarantees                      |
| -------------- | ------------------------------- | --------------------------------------- |
| Range          | `"express": "^4.18.0"`          | Compatible versions, by convention      |
| Exact version  | `"express": "4.18.2"`           | That package only, not its dependencies |
| Lockfile       | `package-lock.json`, `poetry.lock` | The full dependency tree              |
| Content hash   | `integrity: sha512-...`         | The exact bytes                         |

Commit the lockfile and install from it in CI. `npm ci`, for example, installs precisely what the lockfile says and fails if it disagrees with `package.json`, while `npm install` may update it.

The same rule applies beyond language packages. A [container image](#container-image) tag can be moved, so pin a base image by digest. Pin CI actions to a full commit SHA, because a tag such as `v4` can be repointed:

```dockerfile
FROM node:22-slim@sha256:<digest>
```

```yaml
- uses: actions/checkout@<full-commit-sha> # v4
```

Pinning protects the [software supply chain](#software-supply-chain) from unexpected changes, but it also stops security fixes arriving automatically. Tools such as Renovate and Dependabot address this by opening regular pull requests that bump versions and run through [continuous integration](#continuous-integration). A short delay before adopting new releases gives others time to spot a bad one.

_Usage:_

"Nobody touched the repo over the weekend, but the build is red."

"The [Dockerfile](#dockerfile) runs `npm install` with no lockfile, so a transitive package moved. Add dependency pinning and switch to `npm ci`."

### SBOM

An SBOM (software bill of materials) is a machine-readable inventory of the components inside a piece of software. For each package it typically records the name, version, supplier, license, a unique identifier, and how components depend on each other. It works like an ingredient list for a [build artifact](#build-artifact) or [container image](#container-image).

The practical case for one appears when a serious vulnerability is announced in a widely used library. The immediate question is "are we affected, and where?". Without an inventory, engineers search repositories by hand and still miss copies pulled in several levels down or bundled inside a vendor's image. With SBOMs stored for every release, the answer is a query across existing data.

Two standard formats are in common use:

| Format    | Maintained by | Origin and emphasis                       |
| --------- | ------------- | ----------------------------------------- |
| SPDX      | Linux Foundation | Began with license compliance          |
| CycloneDX | OWASP         | Began with application security           |

Tools such as Syft and Trivy can generate either format. Generate the SBOM in the [pipeline](#pipeline) at build time, from the final artifact when possible, because a source repository does not show what the base image added. Store it with the artifact and sign it, so that it can be tied to one specific build as part of the [software supply chain](#software-supply-chain).

An SBOM says a component is present, without saying whether the vulnerable function is ever called. A companion format, VEX (Vulnerability Exploitability eXchange), lets a supplier state whether a product is affected by a given flaw. Generators can also miss statically linked libraries, copied source files, and binaries downloaded during a build.

The list of known vulnerabilities grows after a build, so stored SBOMs should be re-checked regularly, using the same vulnerability data as [image scanning](#image-scanning).

_Usage:_

"A critical CVE just dropped in that XML library. Do we ship it anywhere?"

"Query the SBOMs for every production release. That'll list the affected services without us grepping all the repos."

### Image scanning

Image scanning is the automated inspection of a [container image](#container-image) to find known security problems. A scanner identifies the operating system packages and application libraries in each [image layer](#image-layer), then compares their versions against vulnerability databases, reporting matches as CVEs (public identifiers for known flaws). Trivy, Grype, and the scanners built into many [container registries](#container-registry) are examples.

Scans can run at several points:

| Where                           | Purpose                                   |
| ------------------------------- | ----------------------------------------- |
| Developer machine               | Early feedback before pushing             |
| [Pipeline](#pipeline)       | Block a build that fails the threshold    |
| Registry                        | Scan on push, and rescan stored images    |
| [Kubernetes](#kubernetes) admission | Reject images without a passing result    |
| Running workloads               | Find newly disclosed issues in production |

Rescanning matters because an image that was clean last month can show a critical finding today, when a flaw is disclosed in a package it already contains.

Teams adopting a scanner usually meet alert fatigue first. The initial report on a full-distribution base image can list hundreds of findings, many with no available fix or in code the application never runs. If the pipeline blocks on all of them, developers learn to bypass it. A workable gate fails only on high or critical findings that have a fix available, with time-limited exceptions for the rest.

To reduce findings, use a slim or distroless base through a [multi-stage build](#multi-stage-build), and rebuild regularly. If the base uses [dependency pinning](#dependency-pinning), update the pin, or the rebuild picks up no fixes.

Linux distributions often backport fixes without changing the upstream version number, which can produce false positives in tools that ignore distribution advisories. A clean scan covers only known issues in detected components. It does not find flaws in your own code or zero-day vulnerabilities (flaws not yet public).

_Usage:_

"The scanner reports 312 CVEs and [CI](#continuous-integration) is blocked."

"Most are in the full Debian base. Move to a slim image, then have image scanning fail only on fixable criticals."
