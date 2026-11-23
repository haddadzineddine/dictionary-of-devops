---
description: A Kubernetes status meaning the kernel killed a container for running out of memory, usually at its memory limit.
---

OOMKilled is the status [Kubernetes](./Kubernetes.md) reports when the Linux kernel's out-of-memory (OOM) killer ends a [container](./Container.md). It usually means the container reached the memory limit in its [resource requests and limits](./Resource%20requests%20and%20limits.md), which is enforced by a [cgroup](./cgroup.md). It can also happen when the whole node runs out of memory.

The kill uses `SIGKILL`, which a process cannot catch, so the application cannot log an error or shut down cleanly. The typical symptom is a service that restarts at random with logs that stop without explanation. The evidence is in the [Pod](./Pod.md) status:

```text
Last State:  Terminated
  Reason:    OOMKilled
  Exit Code: 137
```

Exit code 137 means 128 plus signal 9 (`SIGKILL`). It does not prove an OOM kill on its own, because any `SIGKILL` produces it, including one sent when a process ignores a shutdown request. Check for the `OOMKilled` reason as well. Repeated kills usually lead to [CrashLoopBackOff](./CrashLoopBackOff.md).

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
