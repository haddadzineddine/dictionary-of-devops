---
description: A way to make breaking schema or API changes in compatible steps, adding the new form before removing the old.
---

Expand and contract, also called parallel change, is a way to make a breaking change to a database schema or API through a series of compatible steps. You first add the new form alongside the old one, move everything across, and remove the old form only when nothing uses it. Old and new application versions keep working at every step.

The technique exists because deployments are not instantaneous. During a [rolling deployment](./Rolling%20deployment.md), old and new code run at the same time against one database. If a release renames `name` to `full_name` in a single migration, every old instance starts failing with "column does not exist" the moment the migration runs. Reversing the application with a [rollback](./Rollback.md) does not help, because the old code needs a column that is gone.

Renaming that column safely takes several releases:

| Step | Phase    | Change                                         | Safe to roll back? |
| ---- | -------- | ---------------------------------------------- | ------------------ |
| 1    | Expand   | Add nullable `full_name` column                | Yes                |
| 2    | Migrate  | Deploy code that writes to both columns        | Yes                |
| 3    | Migrate  | Backfill existing rows in small batches        | Yes                |
| 4    | Migrate  | Deploy code that reads from `full_name`        | Yes, still dual-writing |
| 5    | Migrate  | Deploy code that writes only `full_name`       | To step 4. Earlier needs a re-sync |
| 6    | Contract | Drop `name` after confirming nothing uses it   | No                 |

Step 6 is the only destructive change and can wait until the team is confident. Verify the backfill first, and check for other readers such as reports and scheduled jobs.

The same pattern applies to APIs: add the new field, migrate clients, then remove the old one. [Continuous deployment](./Continuous%20deployment.md) depends on this pattern, at the cost of more releases and temporary dual-write code.

_Usage:_

"Can I rename this column in today's migration?"

"Not in one step, because the old [pods](./Pod.md) will crash mid-deploy. Use expand and contract: add the new column first and drop the old one next week."
