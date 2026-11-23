---
description: A package manager for Kubernetes that renders templated manifests from values and tracks each install as a release.
---

Helm is a package manager for [Kubernetes](./Kubernetes.md). It bundles the manifests an application needs into a versioned package called a chart, fills in templates using configuration values, applies the result to a cluster, and records each install or upgrade so it can be reversed.

| Term       | Meaning                                                     |
| ---------- | ----------------------------------------------------------- |
| Chart      | A package of templates, defaults, and metadata              |
| Values     | Configuration supplied to the templates                     |
| Template   | A manifest with placeholders, written in Go template syntax |
| Release    | One installed instance of a chart in a cluster              |
| Revision   | A numbered version of a release                             |
| Repository | A place that hosts charts, often an OCI [container registry](./Container%20registry.md) |

A template fragment looks like this:

```yaml
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: api
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

The same chart can then deploy to staging and production with a different values file. Charts carry a [semantic version](./Semantic%20versioning.md), so a team can install third-party software at a known version.

Generating YAML through text templates has known failure modes. Wrong indentation produces invalid manifests, or valid ones with a field in the wrong place. A misspelled key in a values file is usually ignored without any warning, so an override appears to do nothing. Run `helm template` or `helm upgrade --dry-run` to inspect the rendered output before applying it.

A successful `helm upgrade` can also mislead. Without a wait option, Helm reports success once the API server accepts the resources, while the new [Pods](./Pod.md) may be crashing. Use `--wait` with a timeout in [pipelines](./Pipeline.md), and `--rollback-on-failure` to undo a failed upgrade. Helm 3 calls this flag `--atomic`, and Helm 4 renamed it.

`helm rollback` restores an earlier revision's manifests by creating a new revision. It does not undo data changes. Helm stores release history in [Secrets](./Kubernetes%20Secret.md) in the release's namespace by default, so deleting those Secrets makes Helm lose track of the release.

_Usage:_

"I set `replicas: 5` in the values file, and it's still running two."

"The chart reads `replicaCount`, so Helm ignored your key. Run `helm template` and check the output."
