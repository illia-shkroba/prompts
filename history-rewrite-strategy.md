## History Rewrite Strategy

Applies when the user explicitly asks to rewrite existing commits: split,
fold in a fix, reword, reorder, drop. This overrides commit-strategy's
"never squash or amend" rule -- that rule guards against unrequested
rewrites; this file governs requested ones.

**Preconditions:**
1. Working tree and index clean. Do not mix a rewrite with new work.
2. Determine the rewrite base. Commits reachable from a remote branch are
   off-limits: the scope must start after `git merge-base HEAD @{upstream}`.
   No upstream → confirm the whole branch is unpushed with the user.
   Rewriting shared history requires an explicit user override.
3. Create a backup ref: `git branch backup/<branch>-rewrite`. Never delete
   it -- that is the user's call.

**Plan gate:**
4. Show `git log --oneline <base>..HEAD` and the intended end state: which
   commits split into what, which messages change, where fixes land. Wait
   for explicit approval.

**Recipes** -- non-interactive only; `git rebase -i` without a scripted
`GIT_SEQUENCE_EDITOR` is unavailable:

- Fold a change into an old commit: make the edit, stage it,
  `git commit --fixup=<sha>`, then `git rebase --autosquash <sha>~1`
  (git ≥ 2.44; older: `GIT_SEQUENCE_EDITOR=true git rebase -i --autosquash`).
- Stop on a commit for surgery (split, reword):
  `GIT_SEQUENCE_EDITOR="sed -i 's/^pick <sha7>/edit <sha7>/'" git rebase -i <sha7>~1`
  - Split: `git reset HEAD^`, then stage and commit each unit following
    commit-strategy (staged-snapshot verification per unit), then
    `git rebase --continue`.
  - Reword: `git commit --amend -m "<new message>"`, then
    `git rebase --continue`.
- Reorder / drop: `GIT_SEQUENCE_EDITOR` script that rewrites the todo list
  (reorder `pick` lines, change `pick` to `drop`).
- Conflict during rebase: resolve only if the resolution is obvious from
  the plan; otherwise `git rebase --abort` and report.

**Verification:**
5. `git range-diff backup/<branch>-rewrite...HEAD` -- structure changed
   exactly as planned; no unintended message or content drift.
6. `git diff backup/<branch>-rewrite HEAD` -- must be empty unless the plan
   included a content change; then it must show only that change.
7. When commits must each stay green: `git rebase <base> -x "<test cmd>"`
   runs the suite per commit.

**Abort path:**
- Mid-rebase: `git rebase --abort`.
- After a bad finish: `git reset --hard backup/<branch>-rewrite`.
- Never force-push; pushing rewritten history is the user's call.
