---
description: A Git branch that holds one change, such as a feature or bug fix, until it is reviewed and merged into the main branch.
---

A feature branch is a Git branch created from the main branch to hold one change, such as a new endpoint or a bug fix, until it is reviewed and merged back. It lets a developer commit unfinished work without affecting anyone else, and it gives reviewers a single diff to read, usually through a pull request (a request to review and merge a branch).

| Step | Who        | What happens                                                        |
| ---- | ---------- | ------------------------------------------------------------------- |
| 1    | Developer  | Creates the branch from the latest main                             |
| 2    | Developer  | Commits and pushes the change                                       |
| 3    | [CI](./Continuous%20integration.md) | Builds and tests the branch                |
| 4    | Reviewer   | Reads the diff and requests changes or approves                     |
| 5    | Developer  | Updates from main and resolves any conflicts                        |
| 6    | Merge tool | Merges the change and deletes the branch                            |

The common problem is a branch that lives too long. Main moves on while the branch stands still, so the diff grows, reviewers put it off, and the final merge brings conflicts that are tedious and easy to get wrong.

A branch can also pass every check and still break main after merging, because the checks ran against an older copy of main. This is a semantic conflict: Git merges the text cleanly, but one side renamed a function the other side started calling.

Two practices reduce the risk. Keeping branches to a day or two, as in [trunk-based development](./Trunk-based%20development.md), limits how far they can drift. A merge queue, which tests each pull request against the latest main plus the changes ahead of it before merging, catches semantic conflicts before they land. Large work can be split into small merges by hiding incomplete behaviour behind a [feature flag](./Feature%20flag.md).

_Usage:_

"Both PRs were green, so why is main failing?"

"Your feature branch was tested before Sam's rename landed. Git merged them without a conflict, but your code still calls the old function. We should turn on the merge queue."
