---
description: A read-only package of filesystem layers and startup metadata from which containers are created.
---

A [container](./Container.md) image is a read-only package containing everything needed to start one: a filesystem made of [image layers](./Image%20layer.md), plus metadata such as the default command, environment variables, working directory, and user. An image is to a container roughly what a program file is to a running process. One image can start many identical containers.

Images are usually built from a [Dockerfile](./Dockerfile.md) and follow the OCI (Open Container Initiative) format, so an image built with one tool can run on other compatible runtimes. They are stored in a [container registry](./Container%20registry.md) and are a common form of [build artifact](./Build%20artifact.md).

An image can be referred to in two ways:

| Reference | Example                 | Can it change?                       |
| --------- | ----------------------- | ------------------------------------ |
| Tag       | `api:1.4.2`, `api:latest` | Yes, a tag can be moved to a new image |
| Digest    | `api@sha256:9f86d0...`  | No, it is a hash of the content      |

The difference causes a recognisable failure. Two servers both run `api:latest`, but they behave differently because each pulled the tag at a different time and got a different image. The name `latest` has no special meaning. It is only the default tag used when none is specified, and it does not guarantee the newest build. In production, use a unique version tag per build, or a digest when you need an exact guarantee, as described in [dependency pinning](./Dependency%20pinning.md).

One image name can cover several CPU architectures through a multi-platform index, a list pointing to one image per platform. If no entry matches the machine, the pull fails with a "no matching manifest" error.

Images accumulate known vulnerabilities over time as new flaws are disclosed. Rebuild them regularly on an updated base and use [image scanning](./Image%20scanning.md) to find what needs attention.

_Usage:_

"Two nodes are running the same image but only one has the bug."

"Same tag, or same digest? If we deployed `latest`, they may have pulled different builds. Compare the digests."
