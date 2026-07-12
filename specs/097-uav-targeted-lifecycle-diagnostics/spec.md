# Feature Specification: UAV Targeted Lifecycle Diagnostics

**Status**: Complete

## Context

Spec 096 localized the stronger 5% loss boundary to Targeted control and found
two `terminate called without an active exception` events after
`GS_GUI_EXIT rc=0`. Current shutdown joins worker threads before the face thread
that can create them, while command logs do not expose queued, dispatched,
response, timeout, or local-block stages with request correlation.

## User Stories

### User Story 1 - Exit without lifecycle abort (Priority: P1)

Ground Station automation can exit after control success or failure without a
joinable-thread destructor abort.

### User Story 2 - Diagnose Targeted control stages (Priority: P1)

An operator can identify whether Arm/Takeoff/Land was blocked locally, queued,
dispatched, timed out, rejected, or accepted, including request ID and elapsed
time when available.

### User Story 3 - Obtain repeatable MiniNDN evidence (Priority: P1)

An operator runs control-only repetitions at 0% and 5% loss and separately
reports lifecycle aborts, process exit, and command completion.

## Functional Requirements

- **FR-001** Shutdown MUST prevent the face thread from creating worker threads
  after worker join checks have completed.
- **FR-002** Shutdown MUST join every joinable face, authority-refresh, YOLO
  prewarm, decoder, and recording-playback worker exactly once.
- **FR-003** Existing runtime/security behavior MUST remain unchanged.
- **FR-004** Targeted request diagnostics MUST log queued, dispatch-rejected,
  dispatched, response, and timeout stages with provider, service, timestamp,
  request ID when available, and elapsed time for terminal stages.
- **FR-005** UAV command diagnostics MUST log attempt, local-block/busy,
  response, and timeout stages with drone, command, timestamp, reason/status,
  and elapsed time.
- **FR-006** Diagnostics MUST not include tokens, certificates, payload bytes,
  credentials, or private key material.
- **FR-007** The isolation parser MUST report lifecycle abort independently of
  Targeted command completion and launcher return code.
- **FR-008** Tests MUST cover lifecycle-abort parsing, stage diagnostics where
  unit-testable, and existing UAV protocol/campaign behavior.
- **FR-009** The MiniNDN verification MUST run control-only at 0% and 5% loss,
  five repetitions each, without automatic retry or replacement.
- **FR-010** Verification MUST require zero lifecycle abort markers across all
  ten runs, while preserving command failures as measured outcomes.
- **FR-011** Evidence MUST classify each failed command by the latest observed
  stage and must not treat a clean process exit as command success.
- **FR-012** Proposal files MUST NOT be modified.

## Success Criteria

- **SC-001** Affected C++ targets, focused tests, and full Python regressions pass.
- **SC-002** Ten unique control-only MiniNDN runs execute once and remain recorded.
- **SC-003** No run contains either observed lifecycle-abort signature:
  `terminate called without an active exception` or
  `__pthread_tpp_change_priority`.
- **SC-004** Every attempted UAV command has an observable terminal or blocked stage.
- **SC-005** 0% and 5% command completion are reported separately without tuning.
- **SC-006** Post-implementation audit distinguishes fixed lifecycle correctness
  from any remaining network reliability boundary.

## Non-Goals

- Adding command retry, changing timeout, changing Targeted/SVS wire semantics,
  or claiming flight-safety validation.
- Refactoring every detached GUI automation thread.
- Changing video/FEC behavior or proposal artifacts.
