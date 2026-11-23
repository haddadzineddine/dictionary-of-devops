---
description: A release strategy that runs two full environments and switches all traffic from the old version to the new at once.
---

A blue-green deployment is a release strategy that keeps two production environments and moves traffic between them in one switch. "Blue" runs the current version and receives all traffic. "Green" gets the new version and is tested while idle. When it looks correct, a router or load balancer (a component that spreads traffic across servers) points everyone at green.

| Step | Blue            | Green           | What happens                           |
| ---- | --------------- | --------------- | -------------------------------------- |
| 1    | Live            | Idle            | Deploy the new version to green        |
| 2    | Live            | Under test      | Run smoke tests against green directly |
| 3    | Idle            | Live            | Switch traffic to green                |
| 4    | Idle, on standby | Live           | Watch metrics, keep blue running       |
| 5    | Next target     | Live            | Blue becomes the next release target   |

The main benefit is a fast [rollback](./Rollback.md): if green misbehaves, the router points back at blue, with no rebuild or redeploy. Unlike a [rolling deployment](./Rolling%20deployment.md), users do not see a long period with two versions mixed. In [Kubernetes](./Kubernetes.md), one simple form runs two [Deployments](./Kubernetes%20Deployment.md) and changes the label selector on a [Service](./Kubernetes%20Service.md).

Both environments typically share one database, which limits rollback. If green's release drops or renames a column, switching back to blue fails. Schema changes need to work with both versions, using [expand and contract](./Expand%20and%20contract.md).

The switch also has side effects:

- Green starts with cold caches, so latency can jump after the cutover.
- Existing long-lived connections, such as WebSockets, may stay on blue until they are closed.
- DNS-based switching is slow and uneven, because clients cache old answers.
- Background workers in both environments may process the same jobs unless one is paused.

Running two full environments can double compute cost, so many teams create green on demand and remove it afterwards.

_Usage:_

"Green's error rate just doubled after the cutover."

"Flip the load balancer back to blue. That's why we keep it running after a blue-green deployment."
