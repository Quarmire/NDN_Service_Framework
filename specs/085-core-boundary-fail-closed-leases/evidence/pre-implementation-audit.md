# Pre-Implementation Audit

**Date**: 2026-07-10

**Mode**: full, pre-implementation, code-aware, ARS methodology and
devil's-advocate stress test

**Verdict**: CONDITIONAL PASS FOR DESIGN; T001/T004 CLOSED, PRODUCTION EDITS
REMAIN BLOCKED BY T002/T003

## Findings

### HIGH - Target Files Already Contain Uncommitted User Work

`pythonWrapper/ndnsf/service.py`, `pythonWrapper/ndnsf/__init__.py`,
`pythonWrapper/src/ndnsf/_ndnsf.cpp`, Core ServiceUser/Provider sources, Repo
sources, and Targeted tests are dirty before 085. These overlap the planned
migration/binding surface. Implementation must not mix or overwrite them.

Resolution: T001 is a hard entry gate. Assign and commit/stash through a
non-destructive user-approved workflow before T010 or T024. No reset/checkout is
permitted. This condition blocks execution, not the architecture.

Status: RESOLVED. The overlapping work was tested and committed by ownership in
`ab61fbc`, `f919b61`, `91cedb0`, and `f67434e`; see `entry-baseline.md`.

### MEDIUM - External Python API Compatibility Is Not Yet Decided

Moving `ExecutionArtifact*` and `RepoDataPlaneProducer` changes imports for
external users not visible in repository scans.

Resolution: T004 must select a major-version removal or a separately owned,
expiring adapter before export deletion. T009/T030/T031 provide tests, caller
proof, and removal gates.

Status: RESOLVED. The migration is a documented `0.2.0` breaking API move with
no duplicate compatibility implementation; see `python-api-decision.md`.

### MEDIUM - New Authority Path Has No Current Performance Baseline

Existing results cover coordinator/NativeTracer behavior but do not constitute
a matched fail-closed lease-authority campaign.

Resolution: T002 now requires three current coordinator-on 60-second multi-user
baseline runs before treatment; T003 freezes current authority-loss failure;
T034-T036 run matched coordinator-off treatment under parent thresholds.

## Resolved During Audit

1. Added EXECUTING state, provider-local atomic validate-and-activate, finally
   release, and a separate execution hard deadline. This prevents eviction
   while business logic still runs.
2. Added the real C++ NativeTracer provider path. One C++/Python payload contract
   now connects `di-native-provider`, `NativeProviderHandler`, Python user driver,
   generated policies, and MiniNDN evidence.
3. Added provider-issued opaque conflict keys. Core atomically prevents overlap
   without parsing DI model/GPU semantics or trusting requester-selected keys.
4. Added an explicit non-Byzantine crash/loss/delay/restart fault model. Local
   provider safety remains enforced for arbitrary input; global all-provider
   ordering is a trusted authenticated user-transaction property.
5. Justified a separate execution lease lifecycle instead of overloading the
   one-shot GenericAdmissionLease.

## Readiness Scorecard

| Dimension | Result |
|---|---|
| Intent and parent fidelity | PASS |
| Core/DI/Repo ownership | PASS |
| Distributed state and failure behavior | PASS |
| Security and authenticated context | PASS |
| Native/Python real-path wiring | PASS |
| Migration and rollback | CONDITIONAL, T001/T004 |
| Test and MiniNDN rigor | PASS, pending execution |
| Requirements/tasks/traceability | PASS |

## Structural Evidence

- 19 functional requirements, seven success criteria, five user stories.
- 38 dependency-ordered tasks; all stories have tasks.
- 100% FR/SC and task traceability; no unknown task references.
- Spec Kit prerequisites pass; GSD healthy; CodeGraph index current.
- ARS review challenged the state-transition logic, real experimental path,
  resource-conflict premise, fault model, and falsifiable performance evidence;
  the resulting corrections are listed above.

085 may begin T001-T009. It must not start production implementation at T010+
until the HIGH dirty-target finding and T004 API decision are closed.
