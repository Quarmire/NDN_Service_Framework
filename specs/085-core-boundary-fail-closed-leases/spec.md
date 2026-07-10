# Feature Specification: Core Boundary And Fail-Closed Execution Leases

**Feature Branch**: `085-core-boundary-fail-closed-leases`

**Created**: 2026-07-10

**Status**: Planned - Pre-Implementation Audit Required

**Parent**: `specs/084-ndnsf-occam-simplification/`

**Input**: Replace the current coordinator/global-refCount and broken local
fallback with provider-authoritative execution leases. Move DI deployment and
artifact policy, Repo producer policy, and application retry inference out of
the generic Python Core surface without changing Core security or invocation.

## User Scenarios & Testing

### User Story 1 - Authority Loss Fails Closed (Priority: P1)

When the provider lease authority is unavailable, a user receives a typed
rejection/unavailable result within a bounded timeout. No synthetic or
untracked lease is returned.

**Independent Test**: Reproduce the current missing `ExecutionLease` fallback,
then run the treatment with the DI lease service absent and prove typed failure,
zero lease creation, and zero execution.

**Acceptance Scenarios**:

1. Given no lease service response, prepare returns `UNAVAILABLE` before the
   configured timeout and no plan reaches COMMITTED.
2. Given a duplicate prepare/commit/release, the provider returns the original
   idempotent result without double reservation or execution.
3. Given a stale provider epoch or conflicting plan digest, the provider rejects
   the operation with a typed reason.

### User Story 2 - Multi-Provider Commit Is Atomic (Priority: P1)

A user-side planner prepares every selected provider, commits only after all
providers prepare the same plan, and starts execution only after every provider
commits. Partial failure is aborted and eventually cleaned by TTL.

**Independent Test**: Run two users against overlapping providers with delayed,
duplicated, rejected, and lost lease operations; observe zero conflicting
committed role assignments and bounded replan.

**Acceptance Scenarios**:

1. One prepare rejection causes abort on every already-prepared provider.
2. One commit timeout prevents execution; reachable providers receive abort or
   release and unreachable providers expire the lease.
3. Provider restart changes its epoch and invalidates every pre-restart lease.
4. Provider-local eviction is rejected while a committed execution lease pins
   the corresponding opaque resource binding.

### User Story 3 - DI Owns Deployment Policy (Priority: P1)

DI users retain deployment records, execution artifacts, materialization, and
explicit idempotent retry through the DI package rather than generic `ndnsf`.

**Independent Test**: Run migrated NativeTracer/Qwen unit and full-network
workflows while a generic non-DI import contains none of these symbols.

**Acceptance Scenarios**:

1. `ExecutionArtifact`, `ExecutionArtifactSpec`, and `ExecutionContext` are
   imported from `ndnsf_distributed_inference`, not generic `ndnsf`.
2. Deployment publication is descriptive and cannot grant, commit, renew,
   release, or authorize a lease.
3. Automatic retry requires explicit idempotency metadata and never infers
   safety from a reason string alone.

### User Story 4 - Repo Producer Policy Leaves Core (Priority: P2)

Repo callers obtain the Repo data-plane producer from `py_repoclient`; generic
Core retains generic segmented/exact Data helpers only.

**Independent Test**: Existing Repo exact-packet/cache tests pass through the
Repo-owned import and `ndnsf` no longer exports `RepoDataPlaneProducer`.

### User Story 5 - Core Security And Invocation Stay Stable (Priority: P1)

Normal, Targeted, permission, NAC-ABE, token/replay, bootstrap, collaboration,
large-data, status, telemetry, and generic admission behavior remain unchanged.

**Independent Test**: Run the frozen Core baseline and security suite before
and after treatment, plus coordinator-off DI MiniNDN.

## Edge Cases

- Provider crashes after prepare but before commit response.
- User crashes after all providers commit but before execution/release.
- Commit or release response is lost and the request is retransmitted.
- Provider restarts with delayed old operations still in the network.
- Two users use the same request ID under different identities.
- Lease expires during active execution; renewal races with expiry.
- A deployment publication is stale, missing, duplicated, or forged.
- The current dirty worktree overlaps target files.
- A Repo caller still imports the old Core producer path.
- An authenticated user behaves incorrectly through crash, timeout, duplicate,
  or stale state; Byzantine users that deliberately claim a false global commit
  set are outside this child threat model.

## Functional Requirements

- **FR-001**: Core MUST define an application-neutral, versioned execution-lease
  envelope with provider, lease epoch, lease ID, requester, request ID, service,
  plan digest, opaque resource-binding proof bytes, state, expiry, execution
  hard deadline, application-supplied conflict keys, and idempotency key.
- **FR-002**: Core MUST provide a thread-safe provider-local execution lease
  table implementing prepare, commit, atomic validate-and-activate, abort,
  renew, release, expiry, validation, counters, and deterministic typed
  rejection reasons.
- **FR-003**: The table MUST reject stale epochs, conflicting digests/bindings,
  overlapping active conflict keys, invalid transitions, expired leases,
  unknown leases, and identity mismatch.
- **FR-004**: Provider restart MUST create a new epoch; prior-epoch operations
  and leases MUST be rejected and MUST NOT authorize execution.
- **FR-005**: DI MUST expose a user-side transaction that prepares all selected
  providers, commits all, and starts execution only after every provider commits
  the same plan digest and epoch observations remain current.
- **FR-017**: A provider execution handler MUST atomically transition its local
  lease from COMMITTED to EXECUTING before business logic and MUST release it in
  a finally-style completion path; eviction MUST pin both states.
- **FR-006**: Partial prepare/commit MUST trigger bounded abort/release and TTL
  cleanup; partial commit MUST NOT become executable.
- **FR-007**: The current local `GRANTED_LOCAL` fallback and missing
  `ExecutionLease` import path MUST be removed; authority loss MUST fail closed.
- **FR-008**: DI deployment records MUST be descriptive and MUST NOT use global
  refCount as execution or eviction authority.
- **FR-009**: Provider-local eviction MUST consult that provider's active
  committed leases for the opaque resource binding.
- **FR-010**: DI execution artifact types/materialization and application retry
  policy MUST move from `pythonWrapper/ndnsf/service.py` to the DI package.
- **FR-011**: `RepoDataPlaneProducer` MUST move to `py_repoclient` without
  changing Repo storage semantics or selecting the Spec 088 canonical runtime.
- **FR-012**: Coordination envelopes and service remain unchanged in this child;
  DI coordinator isolation/removal belongs to Spec 087.
- **FR-013**: Core normal/Targeted invocation, security, permission, NAC-ABE,
  bootstrap, token/replay, collaboration, and large-data behavior MUST not change.
- **FR-014**: Every wire-visible lease operation MUST use the existing V2
  dynamic service path and existing security/token checks; no new Core wire
  protocol or bypass is allowed.
- **FR-018**: The C++ NativeTracer provider and Python DI provider/client MUST
  use one versioned lease payload contract and the same Core C++ lease table;
  full-network acceptance MUST exercise this path rather than a fake-only adapter.
- **FR-019**: The provider-side DI service MUST derive and issue conflict keys
  from its trusted local worker/GPU slot inventory without trusting requester
  keys or embedding DI semantics in Core; empty keys MUST be rejected for
  exclusive NativeTracer roles.
- **FR-015**: Implementation MUST not begin until overlapping pre-existing dirty
  files have an explicit ownership/commit decision.
- **FR-016**: Every migration and deletion MUST be independently revertible and
  satisfy the parent removal and performance gates.

## Key Entities

- **GenericExecutionLease**: Core provider-owned lease envelope.
- **ProviderExecutionLeaseTable**: Thread-safe provider authority and state machine.
- **LeaseOperationRequest/Response**: DI payload carried by the V2 service.
- **DistributedLeaseTransaction**: User-side prepare/commit/abort orchestration.
- **DeploymentRecord**: Descriptive DI plan/deployment metadata, never authority.
- **BoundaryMigration**: Old symbol, new owner/import, caller set, tests, rollback.

## Success Criteria

- **SC-001**: Authority-unavailable tests return typed failure and create zero
  leases or executions; `GRANTED_LOCAL` and the missing import are absent.
- **SC-002**: Concurrency/failure tests observe zero conflicting committed role
  assignments and zero execution before all providers commit.
- **SC-003**: Restart/stale-epoch, duplicate, partial prepare/commit, atomic
  activation, execution hard-deadline, renewal, release-loss, and eviction-race
  tests all pass.
- **SC-004**: Generic `ndnsf` exports zero DI execution artifact/deployment,
  application retry policy, or Repo producer symbols after migration.
- **SC-005**: Core 199-test baseline, all six security regressions, Core Python,
  DI, and Repo focused tests pass with no new skip.
- **SC-006**: At least three matched 60-second coordinator-off multi-user
  MiniNDN runs satisfy the parent completion/p50/p95 thresholds and report zero
  conflicting commits; logs MUST prove the C++ NativeProviderHandler activated
  and released provider-local leases.
- **SC-007**: Every changed concern has exact caller scans, migration evidence,
  rollback command, and READY removal gate.

## Out Of Scope

- V1/Bloom/permission-table removal (Spec 086).
- Advisory coordinator relocation or semantic-cache isolation (Spec 087).
- Repo runtime/wire selection or persistence migration (Spec 088).
- Stream migration (Spec 089) and typed-envelope epoch removal (Spec 090).
- Proposal slides, NDN-SVS, and local credentials.
- Byzantine distributed commit. Providers enforce local safety; global
  all-provider ordering assumes the authenticated DI user transaction follows
  the protocol and may fail only by crash, loss, delay, duplication, or restart.
