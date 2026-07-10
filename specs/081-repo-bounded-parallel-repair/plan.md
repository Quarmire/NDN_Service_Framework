# Implementation Plan: Bounded Parallel Replica Repair

## Durable Scheduling

Upgrade the Repo SQLite schema to version 8. Add repair-job columns:

```text
available_replicas
missing_replicas
object_priority
object_updated_at_ms
```

`REPAIR_SCAN` derives these values from the catalog summary and refreshes them
for pending/retry jobs. `REPAIR_CLAIM` filters by target, state, lease/backoff,
then orders by:

```text
available_replicas ASC
object_priority DESC
object_updated_at_ms ASC
missing_replicas DESC
attempts ASC
repair_id ASC
```

Legacy jobs receive safe defaults and remain claimable.

## Sidecar Concurrency

1. Main thread runs `REPAIR_SCAN` and claims up to `max_jobs` jobs serially.
2. A bounded `ThreadPoolExecutor` runs only `catalog_repair` for claimed jobs.
3. Main thread consumes completed futures and serially sends
   `REPAIR_COMPLETE` or `REPAIR_FAIL`.
4. Every result logs duration and configured worker count.
5. Default worker count remains 1 outside explicit campaign configuration;
   the treatment campaign uses 3 workers and 6 jobs per scan.

## Quorum Finalization Boundary

Local storage success is a durable receipt, not a global commit decision.
Multi-replica stores publish `STAGED` catalog entries. Once the user validates
W receipts, it sends `FINALIZE_WRITE` with the write intent and receipt set to
the confirmed nodes. A provider validates the tuple/quorum, updates its local
manifest, and publishes AVAILABLE. Staged-only objects are excluded from
repair. Repair-authorized copies of an already finalized source commit locally.

## Safety Invariants

- No concurrent direct use of `request_repo` or sidecar `ServiceUser`.
- At most one active lease owner per repair job.
- Target persistence and validation precede `REPAIR_COMPLETE`.
- A partial write below W never becomes a repair source.
- Worker exceptions always reach durable fail/backoff handling.
- Normal Repo writes share the existing bounded write semaphore, so repair
  cannot create unlimited target pressure.

## Experiment

- Historical reference: accepted Spec 080 workers=1 result, seed 78004.
- Matched baseline: current finalized-write implementation, workers=1,
  max-jobs=6.
- Treatment: same current implementation and 60-second
  topology/load/failure/restart, workers=3, max-jobs=6.
- Primary comparison: repaired strict outage objects and repair coverage.
- Secondary: total repair events, first/last repair latency, request success,
  achieved RPS, write p50/p95, receipt floor, Targeted timeout/fallback counts.

## Validation

- Schema/order/parallelism unit contracts.
- Full Repo Python and focused C++/Targeted/security/concurrency regressions.
- Matched workers=1 and workers=3 MiniNDN campaigns.
- Honest interpretation if parallelism merely moves contention elsewhere.
