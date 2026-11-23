---
description: A server that stores container images and serves them to machines that need to run them.
---

A [container](./Container.md) registry is a server that stores [container images](./Container%20image.md) and serves them to the machines that run them. A [pipeline](./Pipeline.md) pushes a new image after a build, and a host or [Kubernetes](./Kubernetes.md) node pulls it before starting a container. Docker Hub, GitHub Container Registry, Amazon ECR, and Harbor are examples. A general [artifact registry](./Artifact%20registry.md) can often store images alongside other package types.

An image reference names its location in the registry:

| Part       | Example         | Meaning                                  |
| ---------- | --------------- | ---------------------------------------- |
| Registry   | `ghcr.io`       | The server. Docker Hub if omitted        |
| Repository | `acme/api`      | A collection of related images           |
| Tag        | `1.4.2`         | A movable, human-readable label          |
| Digest     | `sha256:9f86...` | The immutable hash of the image manifest |

Together these form `ghcr.io/acme/api:1.4.2`. Pushes and pulls transfer only the [image layers](./Image%20layer.md) the other side lacks, so images that share a base move quickly.

Registry problems usually appear in Kubernetes as `ErrImagePull`, followed by `ImagePullBackOff` while the kubelet (the agent on each node) waits between retries. The usual causes are a mistyped name, a tag that was never pushed, missing credentials for a private repository, or a rate limit. Docker Hub limits pulls, so a cluster that scales up can hit the limit and fail to start workloads that have nothing wrong with them. A pull-through cache, which is a local registry that fetches and stores upstream images on first use, avoids that dependency.

Two settings are worth enabling. Tag immutability, offered by many registries, prevents an existing tag from being pushed over, so `1.4.2` always means the same image. A retention policy removes old images to control storage costs, but it should keep anything currently deployed and enough history for a [rollback](./Rollback.md). Registries can also run [image scanning](./Image%20scanning.md) when an image is pushed.

_Usage:_

"The new [pods](./Pod.md) are all stuck in ImagePullBackOff."

"Check that the tag exists in the container registry. If it does, the node's pull secret has probably expired."
