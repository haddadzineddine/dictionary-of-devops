---
description: An approach where servers and containers are never changed in place. Each change builds a new version and replaces the old.
---

Immutable infrastructure is an approach in which running servers and [containers](./Container.md) are never modified after they are deployed. To change anything, whether an application version, a package, or a setting, you build a new image and replace the old instances with it. The image can be a [container image](./Container%20image.md) or a virtual machine image built with a tool such as Packer.

| Aspect          | Mutable                           | Immutable                         |
| --------------- | --------------------------------- | --------------------------------- |
| Applying a change | Update the running server       | Build a new image and replace it  |
| Server history  | Accumulates over its lifetime     | Exactly what the image contains   |
| Failed update   | Can leave a partial state         | New instance fails, old one remains |
| Going back      | Reverse each change               | Redeploy the previous image       |

The approach addresses the "snowflake server", a machine that has been patched and adjusted by hand for years until nobody can recreate it, and everyone avoids rebooting it. Replacing instances routinely prevents that history from forming and removes most [configuration drift](./Configuration%20drift.md) on the host.

Immutability has requirements. Anything that must survive replacement, such as uploaded files, databases, and logs, has to live outside the instance in external storage or a logging service. Build and deployment must be automated, because a small change now means a full build and rollout. Configuration that varies by environment is supplied at startup, so that one [build artifact](./Build%20artifact.md) works everywhere.

Debugging habits also change. A fix patched onto a live server vanishes at the next deployment or scaling event, so it has to go into the image definition. A [rollback](./Rollback.md) means redeploying the previous image, provided data changes remain compatible. [Kubernetes](./Kubernetes.md) works this way by default: changing a [Pod](./Pod.md) template creates new Pods and removes the old ones.

_Usage:_

"I patched the config on that box last week. Why has the bug come back?"

"We run immutable infrastructure, so autoscaling replaced that instance. The fix needs to go in the image build."
