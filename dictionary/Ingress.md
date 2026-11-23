---
description: A Kubernetes object with HTTP routing rules that send outside traffic to Services, carried out by an ingress controller.
---

An Ingress is a [Kubernetes](./Kubernetes.md) object that defines how HTTP and HTTPS traffic from outside the cluster reaches internal [Services](./Kubernetes%20Service.md), based on hostname and URL path. One public entry point can route `app.example.com/api` to one Service and `app.example.com/` to another, and can terminate TLS (decrypt HTTPS) using a certificate stored in a [Kubernetes Secret](./Kubernetes%20Secret.md).

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
| 503            | The Service has no ready [Pods](./Pod.md)                   |
| 504            | Backend too slow for the proxy's timeout                    |
| Wrong certificate | TLS host or Secret mismatch, or Secret in another namespace |

Exact responses vary by controller. Behaviour beyond basic routing, such as rewrites, rate limits, and timeouts, is usually set through controller-specific annotations, so an Ingress written for one controller may not work the same on another.

The Ingress API is frozen, and new features go to its successor, the Gateway API, which separates infrastructure and routing concerns into distinct objects. Ingress remains supported and widely used.

_Usage:_

"I created the Ingress, but the address column is empty and the domain doesn't resolve."

"Is there a controller in this cluster? The object does nothing until something implements it, and your `ingressClassName` has to match."
