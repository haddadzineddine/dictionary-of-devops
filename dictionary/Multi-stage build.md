---
description: A Dockerfile with several FROM stages, where only selected outputs are copied into a smaller final image.
---

A multi-stage build is a [Dockerfile](./Dockerfile.md) with more than one `FROM` instruction, where each `FROM` starts a separate stage and later stages copy only the files they need from earlier ones. Compilers, package managers, and source code stay in the build stage. The final [container image](./Container%20image.md) contains the application and its runtime needs.

```dockerfile
FROM golang:1.23 AS build
WORKDIR /src
COPY . .
RUN CGO_ENABLED=0 go build -o /app ./cmd/api

FROM gcr.io/distroless/static-debian12
COPY --from=build /app /app
ENTRYPOINT ["/app"]
```

Without this technique, a small service can ship as an image of more than a gigabyte because it includes the entire toolchain used to compile it. Large images are slower to push and pull, which slows deployments and the start of new replicas during scaling. Every extra package also adds findings to [image scanning](./Image%20scanning.md) and gives an attacker more tools to use after a break-in.

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
