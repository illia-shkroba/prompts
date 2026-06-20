## Commit Strategy

**Before writing any code:**
1. Inspect the last 10 commits (or all if fewer; ask user if none). Infer
   commit message style (format, capitalization, verb tense). If commits
   share a common prefix (e.g. `ADO #123:`, `JIRA-456:`), ask the user
   which to use.
2. Propose a commit sequence as a numbered list referencing natural order
   steps (foundational → additive). Wait for explicit approval.

Never squash — user's responsibility.

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
2. Run the project's test suite and linter.
3. Pass → commit. Lint fail → fix inline, re-run. Failures not fixable within
   the current staged unit (e.g. test fail due to missing dependency in another
   unit) → follow the unblocking procedure below.

**Unblocking procedure:**
1. Stash current staged changes: `git stash push --staged`
2. Implement the required fix, touching whatever units are needed
3. Run the test suite and linter — the fix itself must pass before committing
4. Commit the fix with a message that explains the dependency, e.g.:
   "Fix logging framework before `logExternalPlatform` can be used with `mTLS`"
5. Unstash: `git stash pop`. If pop produces conflicts, resolve them, verify
   changes remain functionally correct, then run the test suite and linter.
6. Pass → commit the original logical unit as planned. Fail → repeat.
