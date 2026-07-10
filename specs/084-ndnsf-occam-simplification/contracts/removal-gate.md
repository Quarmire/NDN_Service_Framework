# Removal Gate Contract

Before deleting a mechanism, create a completed record using this template:

```markdown
## <mechanism>

- Current owner:
- Target owner/disposition:
- Repository callers:
- Known external callers:
- External ABI decision:
- Replacement/no-replacement:
- Security invariants:
- Persistence/wire impact:
- Migration commits:
- Focused test commands:
- Module regression commands:
- MiniNDN command and evidence:
- Matched performance result:
- Rollback commit/command:
- Decision: READY | BLOCKED
```

`READY` requires every applicable field. “No callers found” must include the
exact CodeGraph/`rg` query. Network-visible changes require MiniNDN evidence.
Stored-data changes require restart and migration tests.

“Applicable” is determined by this matrix, not by the implementer after seeing
results:

| Change class | Mandatory gate fields |
|---|---|
| Source-only internal deletion | caller scan, focused tests, module regression, rollback |
| Public API or ABI | all source-only fields plus external ABI decision and adapter/version decision |
| Wire name/schema/security | public API fields plus security review, malformed/mixed-version tests, MiniNDN |
| Stored data/schema | wire fields plus migration, rollback-open, restart, and downgrade decision |
| Hot path/performance | relevant fields plus `experiment-gates.md` matched campaign |
| Distributed authority/lease | wire fields plus concurrency, timeout, duplicate, restart, stale epoch, partial failure |

The child feature owner writes the gate. A reviewer who did not implement the
deletion approves READY. `not-applicable` must include a one-sentence reason.
Unknown or disputed classification remains BLOCKED.
