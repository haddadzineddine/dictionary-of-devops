---
description: A read-only set of filesystem changes. Stacked layers form a container image and are cached and shared between images.
---

An image layer is a read-only set of filesystem changes: files added, modified, or deleted relative to the layer below it. A [container image](./Container%20image.md) is an ordered stack of these layers, and a union filesystem merges them into the single directory tree that a [container](./Container.md) sees. In a [Dockerfile](./Dockerfile.md), instructions such as `RUN`, `COPY`, and `ADD` each create a layer.

Layers are identified by a hash of their content, which gives three benefits:

| Benefit       | What happens                                                         |
| ------------- | -------------------------------------------------------------------- |
| Build caching | Unchanged steps reuse an existing layer                              |
| Faster pushes | A [container registry](./Container%20registry.md) receives only the layers it lacks |
| Shared storage | Images with the same base keep one copy on disk                     |

When a container starts, the runtime adds a thin writable layer on top. Changing an existing file copies it into that layer first (copy-on-write), and the layer is deleted with the container.

Because lower layers are read-only, deleting a file in a later layer does not remove it. The later layer records a "whiteout" marker that hides the file, while the bytes remain in the earlier layer. For example, a Dockerfile downloads a 500 MB archive in one `RUN`, deletes it in the next, and the image stays 500 MB larger.

The same mechanism creates a security problem. If a credential was copied in and removed later, anyone who can pull the image can extract the earlier layer and read it. Treat any secret that ever entered a layer as leaked, and rotate it.

To keep layers small, create and delete temporary files within the same `RUN` command, or use a [multi-stage build](./Multi-stage%20build.md). `docker history` shows what each layer added.

_Usage:_

"I added `rm -rf /tmp/build` at the end, but the image didn't shrink."

"That's a new image layer hiding the files. The bytes are still in the earlier one. Do the cleanup in the same `RUN` step."
