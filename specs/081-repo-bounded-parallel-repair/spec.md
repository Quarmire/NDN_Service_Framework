# Feature Specification: Bounded Parallel Replica Repair

**Feature Branch**: `Experimental`

**Created**: 2026-07-10

**Status**: Complete

## Goal

Drain recovered-node repair backlogs faster and in a deterministic risk-aware
order, while preserving durable leases, exact-name validation, normal workload
availability, and an explicit upper bound on repair concurrency.

## User Stories

### US1 - Risk-Aware Repair Order (P1)

Objects with fewer available replicas are repaired before safer objects; ties
prefer higher application priority and then older objects.

### US2 - Bounded Parallel Transfer (P1)

An operator can configure a small repair worker count. Job claim and completion
remain serialized through the sidecar, while independent object transfers run
concurrently within that bound.

### US3 - Measured Backlog Improvement (P1)

A matched MiniNDN failure/restart campaign reports outage repair coverage,
repair throughput, latency, and normal request behavior for single-worker and
bounded-parallel repair.

## Functional Requirements

- **FR-001**: Repair scheduling metadata MUST survive Repo restart.
- **FR-002**: Each durable job MUST record available replica count, missing
  replica count, object priority, and object update epoch.
- **FR-003**: Existing databases MUST migrate in place without losing objects,
  catalog state, or repair jobs.
- **FR-004**: Eligible jobs MUST be ordered by fewer available replicas first,
  higher object priority second, older object update epoch third, then retry
  attempts and stable repair ID.
- **FR-005**: Backoff eligibility (`next_attempt_ms`) MUST remain mandatory.
- **FR-006**: Job claim MUST remain atomic and lease-based.
- **FR-007**: The sidecar MUST claim jobs serially through one `ServiceUser`.
- **FR-008**: Only `catalog_repair` data transfers MAY execute concurrently.
- **FR-009**: `REPAIR_COMPLETE` and `REPAIR_FAIL` MUST be submitted serially by
  the sidecar after worker completion.
- **FR-010**: Repair concurrency MUST be configurable and bounded to 1--8.
- **FR-011**: Maximum jobs per scan MUST be configurable and no smaller than
  the worker count.
- **FR-012**: Worker failures MUST preserve existing durable retry/backoff
  behavior and MUST NOT lose a claimed job silently.
- **FR-013**: Repair logs MUST include duration, worker bound, object, source,
  target, and completion timestamp.
- **FR-014**: The MiniNDN harness MUST expose repair worker/max-job options and
  record them in summary metadata.
- **FR-015**: RF=3/W=QUORUM receipts, Targeted security, exact packet identity,
  and normal read/write behavior MUST remain unchanged.
- **FR-016**: No proposal slides or NDN-SVS changes are allowed.
- **FR-017**: A multi-replica local durable receipt MUST initially publish a
  `STAGED` catalog entry rather than an `AVAILABLE` committed object.
- **FR-018**: After validating at least W receipts, the user MUST send a
  protected `FINALIZE_WRITE` carrying the write intent and receipt set.
- **FR-019**: A Repo MUST validate the finalize tuple and receipt quorum before
  publishing its local generation as `AVAILABLE`.
- **FR-020**: Staged-only generations MUST NOT create repair jobs or be selected
  as repair sources.
- **FR-021**: Authorized repair writes sourced from an already finalized
  catalog generation MAY commit locally without a second quorum finalize.

## Success Criteria

- **SC-001**: Schema migration tests preserve legacy rows and expose scheduling
  columns with schema version 8.
- **SC-002**: Contract tests prove risk/priority/age ordering and backoff.
- **SC-003**: A sidecar test proves active transfers never exceed the configured
  worker count and complete faster than serialized execution for independent
  synthetic jobs.
- **SC-004**: A 60-second MiniNDN workers=3 campaign repairs more strict outage
  objects than the Spec 080 workers=1 baseline, or records an honest negative
  result if the network/control path remains the bottleneck.
- **SC-005**: Every successful write still has at least W validated receipts,
  and normal request success does not regress materially.
- **SC-006**: Repo Python/C++/Targeted/security/concurrency, Spec Kit, CodeGraph,
  and GSD gates pass.
- **SC-007**: A failed W=QUORUM partial write remains staged and cannot be
  resurrected by repair; a receipt-backed finalized write becomes repairable.

## Non-Goals

- No unbounded worker pool or separate repair protocol.
- No consensus, synchronous repair-before-commit, or cross-object transaction.
- No claim that one 60-second campaign defines a production recovery SLO.
