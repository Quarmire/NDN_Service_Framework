# Feature Specification: Targeted Parallel Repo Control Plane

**Feature Branch**: `Experimental`

**Created**: 2026-07-10

**Status**: Complete

## Goal

Reduce NDNSF-REPO write tail latency without weakening confirmed-write,
publisher-ownership, token, reservation, or exact-wire guarantees. Known Repo
providers use NDNSF Targeted invocation, and independent replica operations are
submitted concurrently through one ServiceUser runtime.

## User Stories

### US1 - Reusable Targeted Python API (P1)

Python applications can register a service for normal and targeted invocation
and can issue synchronous or asynchronous Targeted requests to a known
provider. The first call bootstraps one-time token pairs; later calls use the
Targeted fast path and retain all NDNSF security checks.

### US2 - Parallel Confirmed Replica Operations (P1)

A Repo write to RF independent nodes reserves and stores replicas in parallel,
then validates every returned receipt against one WriteIntent. Partial failure
remains visible and retry remains idempotent.

### US3 - Measured Tail-Latency Improvement (P1)

Matched MiniNDN campaigns report per-operation control calls, write latency,
p50/p95/p99, failures, receipts, Targeted submissions, and fallback counts.
Negative or neutral results are retained honestly.

## Functional Requirements

- **FR-001**: Core MUST expose sync and async Targeted request APIs in pybind
  and `ndnsf.ServiceUser`.
- **FR-002**: Python provider registration MUST support
  `NormalAndTargeted` without duplicating handlers or weakening ACK behavior.
- **FR-003**: Targeted requests MUST retain permission, NAC-ABE, one-time token,
  replay, expected-provider, and response-token validation.
- **FR-004**: Targeted token batch size MUST be bounded and configurable, with
  a backward-compatible default.
- **FR-005**: Repo MUST use one ServiceUser and one stable submission thread;
  it MUST NOT create one user runtime per replica.
- **FR-006**: Independent requests to selected replicas MUST be submitted
  asynchronously and awaited under one total deadline.
- **FR-007**: Reservation, release, store, and repair operations MUST preserve
  operation IDs and remain safe under timeout/fallback replay.
- **FR-008**: A committed manifest MUST still contain only validated durable
  receipt owners and meet W.
- **FR-009**: Older NormalOnly providers MUST remain usable through a bounded,
  observable fallback.
- **FR-010**: Replica failures MUST not cancel successful sibling receipts.
- **FR-011**: Metrics MUST distinguish normal, targeted, async, timeout,
  fallback, submitted, completed, and maximum concurrent replica calls.
- **FR-012**: Campaign lifecycle output MUST separate read, write, reservation,
  and store timing.
- **FR-013**: Existing Specs 073-077 tests and exact Data behavior MUST remain
  passing.
- **FR-014**: MiniNDN comparison MUST use the same topology, RF/W, object size,
  random seed, offered load, concurrency, and 60-second window as baseline.
- **FR-015**: Documentation MUST state that Targeted is a known-provider
  optimization, not a bypass of normal service security.

## Success Criteria

- **SC-001**: Sync and async Targeted Python regressions pass with Normal and
  Targeted calls sharing one provider handler.
- **SC-002**: A two-replica fake-runtime test observes at least two outstanding
  replica calls while receipts remain deterministic.
- **SC-003**: No successful ALL write has fewer than RF valid receipts.
- **SC-004**: Matched c16/2-RPS and write-heavy campaigns complete without a
  native crash and preserve machine-readable evidence.
- **SC-005**: Write p95 improves over the Spec 077 matched baseline, or the
  measured non-improvement and cause are documented.
- **SC-006**: Full build, C++ Repo tests, focused Python tests, Spec Kit,
  CodeGraph, GSD, and MiniNDN gates pass.

## Non-Goals

- No distributed transaction or consensus protocol.
- No change to NDN-SVS loss-recovery timing.
- No proposal-slide edits.
- No use of multiple ServiceUser sessions with the same identity.
