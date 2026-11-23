---
description: A Kubernetes object giving a changing set of Pods one stable virtual IP and DNS name, and spreading connections across them.
---

A [Kubernetes](./Kubernetes.md) Service is an object that gives a changing set of [Pods](./Pod.md) one stable address. Pods are replaced often and get new IPs each time, so clients connect to the Service's virtual IP or DNS name instead. A Service named `api` in namespace `shop` resolves at `api.shop.svc.cluster.local`, or `api` from within the same namespace.

A label selector chooses the Pods. Matching Pods are recorded in EndpointSlice objects, and kube-proxy, a component on each node, usually routes new connections to those passing their [readiness probe](./Readiness%20probe.md).

| Type           | Reachable from                  | Typical use                                |
| -------------- | ------------------------------- | ------------------------------------------ |
| `ClusterIP`    | Inside the cluster (default)    | Service-to-service traffic                 |
| `NodePort`     | A port on every node            | Simple external access, testing            |
| `LoadBalancer` | A cloud load balancer           | Exposing one service publicly              |
| `ExternalName` | DNS alias to an outside host    | Pointing at an external system             |

A "headless" Service, with `clusterIP: None`, has no virtual IP. DNS returns the individual Pod addresses, which a [StatefulSet](./StatefulSet.md) uses to give each replica a stable name.

A frequent failure is a Service with no ready endpoints. Connections fail even though Pods are running, usually because the selector does not match the Pod labels or no Pod is ready. Check with `kubectl get endpointslices -l kubernetes.io/service-name=api`. A wrong `targetPort` gives a similar symptom with endpoints present.

Balancing happens per connection, not per request. A client using one long-lived HTTP/2 or gRPC connection sends all its requests to one Pod, so one replica can be overloaded while others sit idle. Client-side balancing or a [service mesh](./Service%20mesh.md) can spread those requests. For HTTP routing from outside the cluster, use [Ingress](./Ingress.md).

_Usage:_

"The pods are healthy but the Service just times out."

"Look at its endpoints. If the list is empty, the selector probably says `app: api` while the pods are labelled `app: api-server`."
