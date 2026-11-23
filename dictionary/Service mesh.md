---
description: An infrastructure layer of proxies that handles encryption, retries, routing, and metrics for traffic between services.
---

A service mesh is an infrastructure layer that handles network communication between services using proxies, so applications do not each implement that logic. Traffic passes through a proxy on the way in and out, and a central component configures all the proxies. Istio and Linkerd are examples.

| Feature         | What the mesh does                                           |
| --------------- | ------------------------------------------------------------ |
| Mutual TLS      | Encrypts traffic and verifies both sides' identities         |
| Traffic routing | Splits by weight, for example for a [canary release](./Canary%20release.md) |
| Reliability     | Applies retries, timeouts, and circuit breaking (cutting off a failing dependency) |
| Telemetry       | Reports request rate, errors, and latency per service        |

A mesh has a data plane, the proxies that carry traffic, and its own control component, separate from the [Kubernetes](./Kubernetes.md) [control plane](./Control%20plane.md). The traditional data plane injects a sidecar proxy [container](./Container.md) into each [Pod](./Pod.md). Newer designs, such as Istio's ambient mode, move much of the work to shared per-node proxies.

A mesh adds a hop to every call, proxy memory and CPU, and another system to upgrade and debug. Adopt one for a specific need, such as required encryption between many services. A few services can often manage with a library and a plain [Service](./Kubernetes%20Service.md).

Sidecars have known startup and shutdown problems. An application that makes a network call before its proxy is ready fails at boot, which can look random. A Job (a workload that runs to completion) can hang because its main container finished while the sidecar kept running. Kubernetes native sidecars, declared as init containers with `restartPolicy: Always`, address both.

Retries also need coordination. If three layers each try a request three times, one failing call can become 27 attempts against a service that is already struggling.

_Usage:_

"The app fails its first DB call on boot, but only since we added the service mesh."

"The sidecar isn't ready when your app starts. Switch to native sidecars so the proxy comes up first."
