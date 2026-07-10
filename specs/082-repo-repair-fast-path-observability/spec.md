# Feature Specification: Repo Repair Fast Path and Observability

**Feature Branch**: `Experimental`

**Created**: 2026-07-10

**Status**: Complete

## Goal

Remove the redundant negative target probe from durable replica repair and make
catalog merge, job creation, claimability, transfer, and completion latency
directly measurable without weakening exact-name, hash, authorization, lease,
or quorum-finalization guarantees.

## User Stories

### US1 - Explain Repair Delay (P1)

An operator can distinguish catalog visibility delay, job/backoff delay,
control-plane delay, and object-transfer delay from sidecar logs and campaign
summary evidence.

### US2 - Avoid Known-Miss Waiting (P1)

A durable repair action whose catalog plan already identifies a missing target
does not wait for a negative `FETCH_PREPARE` selection timeout before copying.

### US3 - Measured Fast Path (P1)

The same workers=3 MiniNDN outage/restart campaign measures whether removing
the probe improves repair count, first-repair latency, and request latency.

## Functional Requirements

- **FR-001**: `REPAIR_SCAN` MUST report created jobs, state counts, target-local
  claimable count, and the earliest future retry epoch.
- **FR-002**: Claimability MUST honor target, state, backoff, and lease rules.
- **FR-003**: The sidecar MUST record scan, claim, transfer, and total cycle
  duration with created, claimable, claimed, completed, and failed counts.
- **FR-004**: Peer delta fetch/merge logs MUST include duration, entry count,
  batch count, peer, target Repo, and timestamp.
- **FR-005**: A catalog-generated durable repair MUST skip the target
  `FETCH_PREPARE` preflight because the action already names a missing target.
- **FR-006**: Source `FETCH_PREPARE`, exact-name segmented retrieval, wire/hash
  validation, repair authorization, and target persistence validation MUST
  remain mandatory.
- **FR-007**: Replaying a repair copy MUST remain safe and idempotent at the
  object/digest level.
- **FR-008**: One `ServiceUser` owner thread MUST remain the control-plane
  boundary; no duplicate same-identity SVS runtime is allowed.
- **FR-009**: Transfer worker count MUST remain bounded and default to one.
- **FR-010**: Campaign summaries MUST include parsed repair-cycle and merge
  telemetry plus the existing failed-write repair invariant.
- **FR-011**: Proposal slides and NDN-SVS MUST NOT be modified.

## Success Criteria

- **SC-001**: Unit tests prove scan counters match durable SQL states and
  backoff/target claimability.
- **SC-002**: A repair-flow test proves no target `FETCH_PREPARE` is issued and
  source prepare plus authorized target store remain.
- **SC-003**: Sidecar tests verify structured cycle metrics and bounded worker
  behavior.
- **SC-004**: A 60-second workers=3 MiniNDN campaign completes 30/30 requests,
  preserves W=2, and reports zero invalid repairs.
- **SC-005**: The campaign records a positive or negative repair-latency result
  against the accepted Spec 081 workers=3 run without rerun bias.
- **SC-006**: Repo Python/C++/Targeted/security/worker, Spec Kit, CodeGraph, and
  GSD gates pass.

## Non-Goals

- No second `ServiceUser` per transfer worker.
- No new repair wire protocol, consensus, or synchronous repair-before-commit.
- No claim that one 60-second campaign defines a production SLO.
