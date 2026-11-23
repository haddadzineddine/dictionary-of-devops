---
description: A Kubernetes controller that changes a workload's replica count based on CPU, memory, or custom metrics.
---

A Horizontal [Pod](./Pod.md) Autoscaler (HPA) is a [Kubernetes](./Kubernetes.md) controller that adjusts the number of Pod replicas in a workload, such as a [Deployment](./Kubernetes%20Deployment.md), according to observed metrics. "Horizontal" means adding or removing replicas, as opposed to giving each one more CPU or memory. By default it re-evaluates every 15 seconds using this calculation:

```text
desired = ceil(current replicas × current metric / target metric)
```

With 4 replicas averaging 90% CPU against a 60% target, the result is `ceil(4 × 90 / 60) = 6`. The controller skips changes when the ratio of current to target is within 10% of 1.0, which prevents constant small adjustments.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api
spec:
  scaleTargetRef: { apiVersion: apps/v1, kind: Deployment, name: api }
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target: { type: Utilization, averageUtilization: 60 }
```

CPU utilisation is calculated against the Pod's CPU request from its [resource requests and limits](./Resource%20requests%20and%20limits.md). If `kubectl get hpa` shows `<unknown>` as the current value and nothing scales, the usual cause is a missing CPU request or a missing metrics-server (the add-on that reports resource usage).

Other common problems:

| Symptom                              | Cause                                         | Fix                                    |
| ------------------------------------ | --------------------------------------------- | -------------------------------------- |
| Replicas reset on each deploy        | Manifest also sets `spec.replicas`            | Remove `replicas` from the manifest    |
| Slow to scale down                   | Default five-minute stabilisation window      | Tune `behavior` if needed              |
| New Pods stay Pending                | Cluster has no spare node capacity            | Add a node autoscaler                  |
| No scaling during overload           | Bottleneck is I/O, so CPU stays low           | Scale on queue depth or request rate   |

The HPA reacts after load has already risen, and new Pods still need time to pull [images](./Container%20image.md) and pass their [readiness probe](./Readiness%20probe.md). For sharp spikes, keep `minReplicas` high enough to absorb the first wave.

_Usage:_

"Traffic tripled and the HPA didn't add a single pod."

"Its target says `<unknown>`. The [container](./Container.md) has no CPU request, so it has nothing to compute utilisation against."
