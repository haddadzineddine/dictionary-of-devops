---
description: The file where Terraform records which real resources it manages, mapping configuration to actual infrastructure IDs.
---

[Terraform](./Terraform.md) state is the data file in which Terraform records the real resources it manages. It maps each block in your configuration, such as `aws_instance.web`, to an actual ID in the provider, such as `i-0abc123`, along with the attributes last seen. Without this mapping, Terraform has no way to know that a server in your account belongs to a given block.

By default state is a local file called `terraform.tfstate`. Teams store it in a remote backend so that everyone works from the same copy:

```hcl
terraform {
  backend "s3" {
    bucket       = "acme-terraform-state"
    key          = "prod/network.tfstate"
    region       = "eu-west-1"
    encrypt      = true
    use_lockfile = true
  }
}
```

Most state incidents fall into a few patterns:

| Problem                  | What you see                                    | Response                                   |
| ------------------------ | ----------------------------------------------- | ------------------------------------------ |
| State lost               | Plan wants to create everything again           | Restore a backup, or import resources      |
| Concurrent applies       | State lock error, or corruption if no locking   | Use locking, and run applies in a [pipeline](./Pipeline.md) |
| Stale lock               | `Error acquiring the state lock` after a crash  | Confirm nothing is running, then `force-unlock` |
| Resource renamed         | Plan shows destroy and create                   | Add a `moved` block                        |
| Resource created by hand | Terraform ignores it                            | Bring it in with an `import` block         |

State often holds secrets. Database passwords and generated keys are stored in it in plain text, even when marked `sensitive` in the configuration, which only hides them from command output. Never commit it to Git. Keep it in an encrypted backend with restricted access, and treat read access to state as access to those credentials.

Enable versioning on the storage bucket, so that a damaged state file can be restored. Avoid hand-editing the file, and use `terraform state` subcommands when you need to change it. Splitting large projects into several smaller states also reduces the impact of a mistake and keeps plans fast.

_Usage:_

"Terraform says it needs to create all 140 resources, but they already exist."

"It's reading empty Terraform state. Check that the backend key is right before anyone applies anything."
