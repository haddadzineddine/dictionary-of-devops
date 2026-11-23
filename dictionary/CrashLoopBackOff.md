---
description: A Kubernetes status meaning a container keeps exiting and the kubelet is waiting longer between each restart attempt.
---

CrashLoopBackOff is a [Kubernetes](./Kubernetes.md) status meaning a [container](./Container.md) keeps exiting and the kubelet (the agent on each node) is waiting before restarting it again. It describes the restart pattern. The cause is whatever made the process exit.

The wait doubles after each failed restart: 10 seconds, 20, 40, and so on, capped at five minutes. The timer resets once a container has run for 10 minutes without a problem. This is why a [Pod](./Pod.md) can look as though nothing is happening for several minutes, then briefly show Running, then return to CrashLoopBackOff.

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
| 137       | `SIGKILL`               | [OOMKilled](./OOMKilled.md), or a forced stop after a failed probe |
| 143       | `SIGTERM`               | Asked to stop, often by a failed [liveness probe](./Liveness%20probe.md) |

Exit code 0 can also produce this status. A [Deployment](./Kubernetes%20Deployment.md) requires `restartPolicy: Always`, so a container that completes successfully is restarted. Use a Job (a workload that runs to completion) for one-off tasks.

If logs are empty, the runtime may have failed to start the command, which `describe` reports as a start error. Check the entrypoint and [image](./Container%20image.md) architecture, or debug with an ephemeral container through `kubectl debug`.

Restarting or deleting the Pod rarely helps, since the replacement hits the same failure.

_Usage:_

"The new release is in CrashLoopBackOff and the logs are empty."

"Run logs with `--previous`, you're looking at the fresh container. And check the exit code in describe."
