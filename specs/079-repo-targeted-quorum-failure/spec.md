# Feature Specification: Targeted Quorum Provider Failure

**Feature Branch**: `Experimental`

**Created**: 2026-07-10

**Status**: Complete

## Goal

Make RF=3, W=QUORUM Repo writes remain correct and bounded when one desired
replica is unavailable. A commit requires two validated durable receipts; the
missing third replica remains visible for later repair and is never claimed as
confirmed.

## User Stories

### US1 - Degraded Quorum Write (P1)

A writer can commit to two reachable replicas when the desired replication
factor is three and one provider fails, while W=ALL still fails.

### US2 - Failure-Aware Targeted Control (P1)

Targeted control records provider success/failure, avoids repeatedly selecting
a provider in cooldown, and bounds fallback within the request deadline.

### US3 - Measured Provider-Loss Evidence (P1)

A matched MiniNDN experiment reports pre-failure, overlapping, and post-failure
success, latency, confirmed replicas, Targeted timeout/fallback counters, and
the exact failure epoch.

## Functional Requirements

- **FR-001**: Desired replication factor and required write acknowledgements
  MUST remain separate values.
- **FR-002**: RF=3/W=QUORUM MUST commit only after at least two validated durable
  receipts.
- **FR-003**: RF=3/W=ALL MUST fail when fewer than three valid receipts exist.
- **FR-004**: Capacity reservation MUST tolerate failed desired replicas only
  when successful reservations meet the write's required acknowledgements.
- **FR-005**: Store calls MUST be sent only to providers with successful active
  reservations when reservation control is enabled.
- **FR-006**: The committed manifest MUST list only validated receipt owners as
  confirmed replicas and MUST retain replicationFactor=3 for later repair.
- **FR-007**: Targeted success, failure, timeout, and fallback MUST update Repo
  provider health and cooldown state.
- **FR-008**: Explicit replica lists MUST not force a provider in active
  cooldown back into the current write when enough healthy replicas remain.
- **FR-009**: Operation IDs, idempotency, publisher ownership, NAC-ABE, tokens,
  replay checks, receipt validation, and repair semantics MUST remain intact.
- **FR-010**: Lifecycle CSV MUST include request start and completion epoch
  timestamps.
- **FR-011**: Failure campaign output MUST separate pre-failure, overlapping,
  and post-failure rows.
- **FR-012**: MiniNDN comparisons MUST use the same topology, seed, offered
  load, concurrency, object size, RF/W, timeout, and 60-second measured window.
- **FR-013**: Existing Specs 073-078 regressions MUST continue to pass.
- **FR-014**: Seed readiness MAY retry only before the measured window; retries
  MUST be bounded, use distinct seed object names, and be recorded.
- **FR-015**: A provider that fails both Targeted and bounded Normal fallback
  MUST enter a stronger cooldown than a provider whose fallback succeeds.

## Success Criteria

- **SC-001**: A contract test commits RF=3/W=QUORUM with exactly two valid
  receipts and reports two confirmed replicas.
- **SC-002**: The equivalent RF=3/W=ALL contract test fails.
- **SC-003**: The provider-loss MiniNDN campaign produces machine-readable
  pre/overlap/post metrics and no fabricated receipt.
- **SC-004**: Post-failure successful writes, if any, contain exactly two or
  three validated receipts and never fewer than two.
- **SC-005**: Full focused build, Repo tests, CodeGraph, Spec Kit, GSD, and
  MiniNDN gates pass.

## Non-Goals

- No consensus protocol or distributed transaction.
- No claim that W=QUORUM immediately restores the missing third replica.
- No proposal-slide changes.
- No change to NDN-SVS retry timing.
