---
description: A custom Kubernetes controller that automates operating an application, driven by custom resource types.
---

A [Kubernetes](./Kubernetes.md) Operator is a custom controller that automates the operation of a specific application by extending the Kubernetes API. It encodes tasks a human administrator would otherwise perform, such as setting up replication, taking backups, handling failover, and running version upgrades.

An Operator has three parts:

| Part                               | Role                                                  |
| ---------------------------------- | ----------------------------------------------------- |
| CustomResourceDefinition (CRD)     | Adds a new object type to the Kubernetes API          |
| Custom resource                    | An instance of that type, stating what you want       |
| Controller                         | Watches those objects and acts to make them true      |

With a PostgreSQL Operator such as CloudNativePG, a database cluster becomes one short manifest:

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: orders-db
spec:
  instances: 3
  storage:
    size: 20Gi
```

The controller creates the [Pods](./Pod.md), storage, and [Services](./Kubernetes%20Service.md), configures replication, and promotes a replica if the primary fails. It runs a reconcile loop, the same pattern built-in controllers use: read the desired state, inspect the actual state, and act on the difference.

Operators are useful for stateful software, where a [StatefulSet](./StatefulSet.md) supplies identity and storage but knows nothing about the application's failover process. For a stateless app, a [Deployment](./Kubernetes%20Deployment.md) or [Helm](./Helm.md) chart is usually enough.

An Operator is extra software you must run and trust. It often holds broad cluster permissions, its CRDs need upgrading alongside it, and a bug in it acts automatically on production systems. If it stops running, nothing reconciles, so failover and repair stop too.

A recognisable failure comes from removing an Operator before its resources. Custom resources often carry a finalizer, a marker that blocks deletion until a controller finishes cleanup. With the controller gone, nothing clears the marker, and `kubectl delete` hangs with the object stuck in `Terminating`. Delete custom resources first, then uninstall the Operator. Removing a finalizer by hand skips that cleanup and can leave external resources behind.

_Usage:_

"This custom resource has been stuck in Terminating since yesterday."

"Its finalizer is waiting for the Kubernetes Operator, and someone uninstalled it first. Reinstall it so that it can finish the cleanup."
