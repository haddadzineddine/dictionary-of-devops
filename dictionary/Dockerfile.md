---
description: A text file of ordered instructions that a builder executes to produce a container image.
---

A Dockerfile is a text file of ordered instructions that a builder executes to produce a [container image](./Container%20image.md). Each instruction starts from the result of the previous one, and those that change files produce a new [image layer](./Image%20layer.md).

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
| `ENTRYPOINT` | Start      | Sets the executable for the [container](./Container.md) |
| `CMD`        | Start      | Sets default arguments or a default command |

Instruction order controls build speed. Builders reuse cached layers until they reach a step whose input changed, then rerun everything after it. If `COPY . .` comes before the dependency install, editing one source file reinstalls every package, and a build that should take seconds takes minutes. Copying the lockfile first, as above, keeps the install step cached until dependencies change.

Three other mistakes are common. Without a `.dockerignore` file, the build context can include `node_modules`, `.git`, and local `.env` files, and `COPY . .` puts them in the image. Passing a password through `ARG` or `ENV` can leave it readable in the image's metadata or history, so use the builder's secret mount feature. Containers run as root unless the image or runtime sets another user.

The form of `CMD` affects shutdown. `CMD node server.js` runs under a shell, which may not forward the stop signal to Node, so the container hangs until it is force-killed. The JSON form runs the process directly. To keep build tools out of the result, use a [multi-stage build](./Multi-stage%20build.md).

_Usage:_

"Every [CI](./Continuous%20integration.md) build reinstalls all our dependencies, even for a typo fix."

"Your Dockerfile copies the whole repo before `npm ci`. Copy the lockfile first so the install layer stays cached."
