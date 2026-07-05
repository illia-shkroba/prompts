## Copy-First Strategy

Applies to new units of code (class, function, test, config block). New code
must read as if written by the author of its closest existing neighbor.
Novel structure requires justification; copied structure does not.

**Before writing code:**
1. Find 1–3 analogs — existing units doing the closest job (same data
   source, base class, lifecycle hook, layer). Proximity ranking: same
   file > same module > same package > rest of repo.
2. Propose the primary analog ("Modeling `NewThing` on `Analog`" plus one
   sentence why); if none exists, say so and propose the novel structure.
   If the analog contains patterns marked deprecated, legacy, or being
   phased out, flag them now instead of copying them. Wait for explicit
   approval.
3. Read the approved analog fully: helpers, error handling, logging, tests.

**While writing:**
4. Draft as a copy of the analog mutated minimally. Preserve its naming,
   control flow (how it branches, iterates, exits, propagates absence or
   failure), error and incident handling, logging, doc-comment style,
   member ordering. New semantics change the inside of the structure, not
   the shape around it.
5. Failure-handling parity: where the analog reports missing
   sub-structures, the new code reports the equivalent — even if the task
   didn't mention it. Silent return where the analog records an incident is
   a bug.

**Tests:**
6. Copy the shape of the neighbors' tests: same fixtures, same iteration,
   same assertion source. If they loop over all fixture documents, loop —
   don't cherry-pick one and hardcode its values.
7. Expected values live where the suite keeps them: extend fixture/golden
   files with the new expected data. Inline literals only when the suite
   has no fixture mechanism for that data.
8. A new field usually propagates to every artifact mirroring the output:
   fixtures, golden files, factories, snapshots. Find the commit that added
   the previous similar field; its diff is the propagation checklist.

**Before presenting the diff:**
9. Diff the new unit against its analog: every difference must trace to a
   requirement (different semantics or failure modes), never taste. Fix
   differences explainable only by "different author, different time".
   List the deviations you kept, with reasons, in the summary.

Example — adding an extraction step next to this existing one:

```python
class InvoiceLineItems(ExtractionStep):
    def get(self, document, *, into):
        if (payload := document.get("invoice-data")) is not None:
            contents = json.loads(payload.contents)
        else:
            log_incident(Incidents.SOURCE_DATA_MISSING)
        ...
```

Wrong — novel shape, silent failure, renamed variable:

```python
class InvoiceAttachments(ExtractionStep):
    def get(self, document, *, into):
        data = document.get("invoice-data")
        if data is None:
            return
        attachments = json.loads(data.contents).get("attachments") or []
        ...
```

Right — the analog's shape, new semantics inside:

```python
class InvoiceAttachments(ExtractionStep):
    def get(self, document, *, into):
        if (payload := document.get("invoice-data")) is not None:
            contents = json.loads(payload.contents)
            attachments = contents.get("attachments") or []
        else:
            log_incident(Incidents.SOURCE_DATA_MISSING)
        ...
```
