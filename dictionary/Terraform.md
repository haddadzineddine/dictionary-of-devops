---
description: A declarative infrastructure as code tool that compares configuration with recorded state and applies the difference.
---

Terraform is a declarative [infrastructure as code](./Infrastructure%20as%20code.md) tool from HashiCorp. You describe resources in HCL (HashiCorp Configuration Language), and Terraform works out what to create, change, or delete to reach that description. Plugins called providers connect it to AWS, Google Cloud, Azure, Cloudflare, and many other APIs. OpenTofu is an open-source fork with a largely compatible workflow.

```hcl
resource "aws_s3_bucket" "logs" {
  bucket = "acme-app-logs"

  lifecycle {
    prevent_destroy = true
  }
}
```

| Command             | What it does                                                  |
| ------------------- | ------------------------------------------------------------- |
| `terraform init`    | Downloads providers and connects to the state backend         |
| `terraform plan`    | Shows the changes it would make, without making them          |
| `terraform apply`   | Makes the changes after confirmation                          |
| `terraform destroy` | Deletes everything the configuration manages                  |

To build a plan, Terraform compares three things: your configuration, its record of what it manages in [Terraform state](./Terraform%20state.md), and the real resources as reported by the provider. Differences found in the real resources are [configuration drift](./Configuration%20drift.md).

A plan can propose destroying a database after a small edit. Some attributes cannot be changed on an existing resource, so changing one means deleting and recreating it. The plan marks this as `-/+` with `# forces replacement` beside the attribute. Renaming a resource block has a similar effect, because Terraform sees one resource removed and another added. A `moved` block tells it that the two are the same.

Read every plan before applying, especially lines containing `destroy` or `replace`. Apply the saved plan with `terraform plan -out=tfplan` followed by `terraform apply tfplan`, so that what was reviewed is what runs. Use `prevent_destroy` on resources that hold data. It makes any plan that would destroy the resource fail, although removing the block from the configuration removes that protection.

_Usage:_

"Terraform wants to replace the production database. I only changed one setting."

"That setting forces replacement. Stop before applying, and find a way to migrate it without recreating the instance."
