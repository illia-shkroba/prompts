## Commit Strategy

**Before writing any code:**
1. Inspect the last 10 commits (or all if fewer; ask user if none). Infer
   commit message style (format, capitalization, verb tense). If commits
   share a common prefix (e.g. `ADO #123:`, `JIRA-456:`), ask the user
   which ticket/prefix value applies to this work.
2. Propose a commit sequence as a numbered list referencing natural order
   steps (foundational → additive). Wait for explicit approval.

Natural commit order (foundational → additive):
1. Dependency bumps (must precede code that uses them)
2. Data structures / types / config models
3. DB migrations
4. Wiring — integrate types/config into existing structures/call sites
   (compile-time stubs acceptable, not full logic)
5. Core implementation logic
6. Logging / observability
7. Tests
8. CI/CD config / docs

Example plan for an mTLS feature:
- Commit 1: "Add `go-mtls` v2.3 dependency"
- Commit 2: "Create `MtlsConfig`"
- Commit 3: "Add `mtls_config` column to `sessions` table"
- Commit 4: "Add `mTLS` field to `Tier` with `MtlsConfig`"
- Commit 5: "Implement `mTLS` handling during session initiation"
- Commit 6: "Add logging for `mTLS` handling"
- Commit 7: "Add tests for `mTLS`"
- Commit 8: "Add `mTLS` manual testing procedure to the docs"

**Before each commit:**
1. Stage only changes belonging to that logical unit. Same file may appear
   in multiple commits — split by logical concern, not by file.
2. Verify the staged snapshot, not the working tree:
   `git stash push --keep-index --include-untracked`, run the test suite
   and linter, then `git stash pop`. Pop conflict → resolve, re-verify.
3. Pass → commit. Lint fail → fix, re-stage, re-verify. Test failure caused
   by a later planned unit → the plan is misordered: reorder or merge units
   and continue (re-propose the sequence if it changes materially). Failure
   requiring a fix outside the plan → unblocking procedure below.

Never squash or amend already-created commits; squashing is the user's call.

**Unblocking procedure** (for newly discovered fixes outside the plan):
1. Park the current unit: `git stash push --staged`
2. Implement the fix, stage it, and verify its staged snapshot as above.
   The fix itself must pass before committing.
3. Commit the fix with a message that explains the dependency, e.g.:
   "Fix logging framework before `logExternalPlatform` can be used with `mTLS`"
4. Unstash: `git stash pop`. Changes may return unstaged; if pop produces
   conflicts, resolve them and verify changes remain functionally correct.
5. Re-stage the original unit, verify its staged snapshot, and commit as
   planned. Fail → repeat.
