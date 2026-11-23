---
description: A branching model where everyone merges small changes into one shared branch at least daily, keeping integration problems small.
---

Trunk-based development is a branching model in which all developers merge small changes into one shared branch, called the trunk (usually `main`), at least once a day. Work happens directly on the trunk or on a [feature branch](./Feature%20branch.md) that lives for a day or two. The trunk is kept in a state that could be released at any time.

The model addresses a familiar failure. Two developers work separately for three weeks, each with passing tests, then spend days resolving conflicts because both changed the same modules. Each branch worked alone, but the combination was never tested until the merge. Merging daily keeps each conflict small, and it makes [continuous integration](./Continuous%20integration.md) possible because there is one line of code to integrate against.

| Aspect          | Long-lived branches           | Trunk-based development       |
| --------------- | ----------------------------- | ----------------------------- |
| Branch lifetime | Weeks or months               | Hours to a couple of days     |
| Merge size      | Large                         | Small                         |
| Conflicts       | Found late, in bulk           | Found early, one at a time    |
| Unfinished work | Hidden on the branch          | Merged, but switched off      |

Unfinished work is merged behind a [feature flag](./Feature%20flag.md), which keeps the code inactive in production. Another option is branch by abstraction, where a new implementation is built behind an existing interface and swapped in once ready. Database changes follow [expand and contract](./Expand%20and%20contract.md) for the same reason: each small step must be safe to ship alone.

The model depends on supporting habits. A fast automated [pipeline](./Pipeline.md) must run on every change, reviews must happen within hours, and a broken trunk must be fixed or reverted before other work lands. Without those habits, frequent merges spread breakage to everyone.

_Usage:_

"My branch is three weeks old and the rebase has forty conflicts."

"That's what trunk-based development avoids. Split it up, merge the refactor today, and put the new checkout behind a flag."
