---
description: Defining servers, networks, and other infrastructure in version-controlled files that tools apply automatically.
---

Infrastructure as code (IaC) is the practice of defining servers, networks, databases, and permissions in version-controlled files, then using a tool to create and update them. The files replace manual work in a cloud console. A change to infrastructure becomes a commit that can be reviewed, tested, and reverted.

It addresses a familiar situation: production was built by hand over several years, nobody knows why one firewall rule exists, and staging differs in ways no one can list. When someone has to rebuild after a failure, the only documentation is the running system. With IaC, the definition is written down and can be run again.

IaC tools take one of two approaches:

| Approach    | You write                  | Tool's job                              | Examples                        |
| ----------- | -------------------------- | --------------------------------------- | ------------------------------- |
| Declarative | The desired end state      | Work out the changes needed             | [Terraform](./Terraform.md), CloudFormation |
| Imperative  | The steps to perform       | Run those steps in order                | Shell scripts, cloud SDK calls  |

Declarative tools are the usual choice because the same file can be applied repeatedly. If the infrastructure already matches, nothing happens. An imperative script that creates a server will often create a second one when it runs again, unless it is written to check first.

A typical workflow puts changes through a [pipeline](./Pipeline.md): open a pull request, generate a preview of what will change, have someone review that preview, and apply after merging. The preview matters, because a small edit can have a large effect, such as replacing a database. [Policy as code](./Policy%20as%20code.md) can check the preview automatically.

IaC only works while the code stays authoritative. Emergency edits in the console create [configuration drift](./Configuration%20drift.md), and the next apply may silently undo them. Credentials should also stay out of these files and come from [secrets management](./Secrets%20management.md).

_Usage:_

"How long would it take to recreate the staging VPC if someone deleted it?"

"Days, since it was all clicked together. We should bring it under infrastructure as code before we need to find out."
