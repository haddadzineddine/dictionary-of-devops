---
description: A Kubernetes object for small sensitive values. Its data is base64-encoded and is not encrypted by default.
---

A [Kubernetes](./Kubernetes.md) Secret is an object for holding small pieces of sensitive data, such as passwords, API tokens, and TLS certificates, so they stay out of [container images](./Container%20image.md) and [Pod](./Pod.md) specs. Pods consume Secrets as environment variables or mounted files. The size limit is 1 MiB.

Secret values are base64-encoded, and base64 is a reversible text format that provides no protection. Anyone who can read the object can decode it with one command:

```sh
echo 'cGFzc3dvcmQ=' | base64 --decode
```

By default the API server also stores Secrets unencrypted in etcd, the cluster's database. A frequent mistake is committing a Secret manifest to Git because the values look scrambled. Rotate any value exposed this way.

Protecting Secrets takes several separate measures:

| Measure                | What it protects against                              |
| ---------------------- | ----------------------------------------------------- |
| Encryption at rest     | Someone reading etcd or its backups                   |
| RBAC (role-based access control) | Users and workloads reading Secrets via the API |
| Namespace separation   | Workloads in other namespaces mounting them           |
| External secret store  | Long-lived values living only in the cluster          |

Access rules have a gap that is easy to miss. Anyone allowed to create a Pod in a namespace can mount any Secret in that namespace and read it, even without permission to `get` Secrets directly. For stronger control, use dedicated [secrets management](./Secrets%20management.md) and sync or inject values at runtime.

Encoding also causes a login failure. `echo "hunter2" | base64` includes a trailing newline, so the application sends a password ending in `\n` and authentication fails. Use `echo -n`, `kubectl create secret`, or the `stringData` field, which accepts plain text.

Updates behave like those of a [ConfigMap](./ConfigMap.md): environment variables keep the old value until the [container](./Container.md) restarts.

_Usage:_

"It's fine to commit this Secret YAML, the password is encoded."

"That's base64, so anyone can decode it. Kubernetes doesn't encrypt it for you. Rotate that password and keep it in Vault."
