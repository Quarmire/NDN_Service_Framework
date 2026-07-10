# Implementation Plan: NDNSF Occam Simplification

**Branch**: `084-ndnsf-occam-simplification` | **Date**: 2026-07-10 | **Spec**: [spec.md](spec.md)

## Summary

Simplify NDNSF by enforcing one owner, one canonical contract, and one runtime
implementation per concern. Spec 084 is the umbrella program: it freezes
cross-cutting invariants and creates audited child features for implementation.
The migration is incremental and evidence-gated:

1. remove unsafe behavior before removing optional coordination;
2. restore Core/application ownership boundaries;
3. retire dead V1 APIs and compatibility overloads;
4. isolate optional DI research features;
5. select the Repo protocol/runtime through an ADR, then converge by parity slices;
6. converge stream state on Core C++;
7. end typed/legacy dual encoding after a bounded compatibility epoch.

No phase may weaken security, exclusive-resource admission, plan atomicity,
exact-name Data semantics, Repo quorum/repair, or UAV flight safety.

## Technical Context

**Language/Version**: C++17, Python 3.8+ on Ubuntu 20.04

**Primary Dependencies**: ndn-cxx, NFD, ndn-svs, NAC-ABE, pybind11, SQLite,
MiniNDN, Tk, MAVLink/H264 support used by UAV

**Storage**: SQLite-authoritative Repo storage with bounded memory cache; files
for model/runtime artifacts and experiment evidence

**Testing**: C++ unit tests, Python unittest/pytest-style scripts, shell
regressions, MiniNDN 60-second matched campaigns

**Target Platform**: Ubuntu 20.04 development and MiniNDN environments

**Project Type**: C++ framework with Python bindings and three application
projects: DI, Repo, UAV

**Performance Goals**: Meet `contracts/experiment-gates.md`: correctness and
security remain exact, completion decreases by at most 0.5 percentage points,
and median matched p50/p95 remain within 110% unless a child spec freezes a
different threshold before treatment

**Constraints**: Preserve user dirty changes; no proposal slides; no NDN-SVS
changes; no security bypass; one independently revertible concern per commit

**Scale/Scope**: Program governance across Core, DI, Repo, and UAV. Production
changes are partitioned into child specs 085-090.

## Constitution Check

- **Canonical Dynamic Runtime**: PASS. Target is V2 unified `serviceName`; V1
  split-name and Bloom-filter paths are removed.
- **Security Is Part Of The Data Path**: PASS. Security behavior is protected by
  explicit non-regression gates.
- **CodeGraph First**: PASS. Current index is up to date; callers and blast
  radius were inspected before planning.
- **Spec-Driven Durable Work**: PASS WITH CHILD GATE. Spec 084 contains program
  requirements and contracts; child specs own implementation plans and tasks.
- **Verify With The Right Scope**: PASS. Core and application regressions are
  paired with MiniNDN gates for network-visible changes.
- **Dirty Worktree Safety**: PASS WITH ACTION. Implementation starts with a
  path-level ownership manifest and must not fold existing Repo work into
  cleanup commits.

## Target Architecture

```text
NDNSF Core C++
  security + bootstrap
  V2 normal/Targeted invocation
  collaboration transport and exact-name large Data
  continuous StreamInfo/StreamChunk + one reorder/adaptive state engine
  generic discovery, capability/runtime/network hints
  generic operation status, rejection reason, admission/execution lease envelope

Python ndnsf binding
  thin binding/adaptation over Core C++
  no DI deployment manager
  no Repo producer type
  no semantic cache
  no app-level retry inference

NDNSF-DI
  user-side planner and bounded replan
  provider inventory/residency and execution runtime
  deployment manager + execution artifact materialization
  provider admission and plan prepare/commit/abort
  optional advisory coordinator plugin
  exact forward cache
  optional experimental semantic cache

DI distributed authority
  user owns descriptive plan/deployment record
  each provider owns leases for its local resources
  prepare all -> commit all -> execute; otherwise abort/replan
  provider restart changes lease epoch; stale leases are rejected
  no global refCount and no synthetic local lease success

NDNSF-DistributedRepo
  one public service contract selected by ADR
  one authoritative storage/catalog/repair runtime selected by ADR
  exact signed Data wire preservation
  SQLite authority + bounded hot cache
  quorum, tombstone, anti-entropy, repair
  private replication protocol hidden and authorization-separated from clients

NDNSF-UAV-APP
  MAVLink, mission, parameter, preflight, operator authority
  H264 framing, FEC codec, ROI policy and UI
  consumes Core stream reorder/health/adaptive state
```

## Ownership And Disposition Matrix

| Mechanism | Current location | Target owner | Disposition |
|---|---|---|---|
| V2 normal/Targeted invocation | Core | Core | Keep |
| V1 split names + Bloom filter | Core | None | Remove after ABI gate |
| Legacy ACK/local handler overloads | Core | None/adapter | Remove if caller scan remains empty |
| DI deployment lifecycle methods | Python Core wrapper | DI | Move |
| ExecutionArtifactSpec/materializer helpers | Python Core wrapper | DI | Move; keep generic large-data refs in Core |
| RepoDataPlaneProducer | Python Core wrapper/binding | Repo | Move/rename into Repo package |
| Generic retry-by-error-string | Python Core wrapper | DI | Move; require idempotency |
| Coordination envelope/service | Core | DI experimental | Remove from Core after DI migration |
| Provider admission + generic lease envelope | Core | Core | Keep |
| DI prepare/commit/abort and fragment residency | DI | DI | Keep |
| Semantic service cache | DI public runtime | DI experimental | Isolate, default off |
| Exact Forward Cache | DI provider | DI | Keep |
| Planner placeholders without handlers | DI | None | Remove |
| Repo C++ and Python storage engines | Repo + DI package | Repo | Freeze parity, decide by ADR, converge to one runtime |
| STORE/raw payload convenience | Repo wire API | Repo client adapter | Remove as independent provider model |
| Packet batch/pull/finalize operations | Repo public surface | Repo private replication protocol | Hide, do not delete behavior |
| Typed + flat legacy ACK/status | All | Core typed contracts | Migrate, then remove flat fields |
| C++/Python/UAV stream reorder logic | Core + UAV | Core C++ | Converge; retain UAV codec policy |
| Operator authority lease | UAV | UAV | Keep; domain safety, not generic admission |

## Migration Strategy

### Program Phase 0 - Baseline And Child Creation

- Freeze exact caller and public-symbol inventories.
- Record dirty-file ownership and canonical test commands/results.
- Add removal-gate automation and freeze experiment thresholds.
- Capture the current broken/untracked lease fallback with a regression.
- Create and audit child specs 085-090 without changing production code.

### Child 085 - Restore Core Boundary And Lease Safety

- Implement the provider-authoritative state machine in
  `contracts/di-lease-authority.md` and remove synthetic local success.
- Move DI-owned `DeploymentManager`, `ExecutionArtifactSpec`, and retry policy
  modules using existing generic Core APIs.
- Move Repo data-plane producer bindings behind the Repo package.
- Migrate all callers before removing Core exports.
- Split `pythonWrapper/ndnsf/service.py` by generic concern only after behavior
  moves; file splitting alone is not acceptance.

### Child 086 - Retire Legacy Invocation

- Add compile-time deprecation/removal manifest and external ABI decision.
- Introduce the unified service-authorization table and migrate current V2
  permission callers before deleting split-name/token storage.
- Remove V1 request generation/parsing, BloomFilter code/build entries, old
  Direct terminology, and only callbacks/overloads proven dead by symbol scans.
- Keep no aliases unless the ABI decision explicitly requires a separately
  packaged adapter.

### Child 087 - Simplify DI Scheduling Surface

- Make pure user-side planning + provider admission the only default.
- Move advisory coordinator implementation, envelopes, tests, and CLI flags to
  `experimental/advisory_coordination` or delete them if no research consumer is
  retained.
- Replace coordinator authority and global refCount with descriptive deployment
  records plus provider-local leases; publication never authorizes execution.
- Remove planner placeholders and isolate semantic cache experiments.

### Child 088 - Decide And Converge Repo

- Freeze black-box exact-packet, persistence, HA, repair, and catalog fixtures.
- Apply `contracts/repo-decision-gate.md` and approve an ADR before migration.
- Incrementally implement missing parity in the selected runtime using the same
  SQLite schema or a tested versioned migration.
- Keep the non-authoritative language layer thin after parity passes.
- Delete duplicate storage/catalog/repair implementation only after parity gates.
- Collapse the external operation list; retain private internal operations.
- Enforce internal operation authorization and negative client tests.

### Child 089 - Converge Stream State

- Freeze `contracts/stream-parity.md`, then extend C++ stream primitives only
  for generic behavior proven necessary by the UAV path.
- Bind those primitives to Python.
- Replace UAV generic pending/reorder/gap/adaptive state with Core objects.
- Preserve H264/FEC/ROI/frame/decode policy and exact-name static-object path.

### Child 090 - End Dual Encoding

- Inventory aliases separately from domain state, then add schema version and
  conflict counters.
- Run one bounded mixed-version compatibility epoch.
- Switch producers to typed-only, then remove legacy parsers, fixtures, tests,
  and docs.

## Removal Gate

Every deletion must satisfy the change-class matrix in
`contracts/removal-gate.md` and all applicable fields below:

```text
mechanism
owner_before / owner_after
repository caller scan
external ABI decision
replacement or explicit no-replacement rationale
security invariant impact
persistence/wire compatibility impact
focused tests
module regressions
MiniNDN command when network-visible
matched before/after metrics when performance-sensitive
rollback commit
```

Deletion is blocked by any unknown external caller, unexplained security change,
missing persistence migration, failed regression, or violation of
`contracts/experiment-gates.md`.

## Verification Strategy

### Core Gate

- C++ build and unit suite.
- Generic dynamic API normal and Targeted tests.
- HELLO auth/ACK/custom-selection/NAC-ABE/token-negative regressions.
- certificate bootstrap and negative-ACK early-stop tests.
- public symbol and forbidden-string scans.

### DI Gate

- runtime_v1, planner, provider inventory, cache, GUI/headless tests.
- NativeTracer full-network and Qwen MiniNDN smoke.
- multi-user lease conflict and bounded-replan campaign with coordinator off.
- optional advisory experiment separately, never as a correctness prerequisite.

### Repo Gate

- exact packet, tiered cache, packet consumer/failover tests.
- HA concurrency, Targeted quorum failure, recovery, repair, worker, and catalog
  merge tests.
- ADR evidence covers semantics, security, operations, maintainability,
  persistence, and performance; one benchmark cannot choose the architecture.
- matched campaigns follow `contracts/experiment-gates.md`: 30/30 per run,
  W=2, zero invalid finalized replicas, expected repair coverage.

### UAV Gate

- protocol and operational-layer unit tests.
- stream stale-session, duplicate, gap, reorder, FEC and backlog tests.
- MiniNDN live video loss campaign.
- mission/preflight/operator-authority regressions.

## Commit And Rollback Policy

- One ownership move or removal concern per commit.
- Tests precede implementation where behavior changes.
- Migration commits land before deletion commits.
- Generated results and unrelated dirty files are excluded.
- Each phase records its pre-change tag/commit and exact rollback command.
- No history rewrite is required by this plan.

## Project Structure

```text
specs/084-ndnsf-occam-simplification/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   ├── ownership-matrix.md
│   ├── child-feature-map.md
│   ├── di-lease-authority.md
│   ├── experiment-gates.md
│   ├── permission-v2-migration.md
│   ├── repo-decision-gate.md
│   ├── removal-gate.md
│   ├── regression-matrix.md
│   └── stream-parity.md
└── tasks.md

ndn-service-framework/                 # generic C++ Core only
pythonWrapper/ndnsf/                   # thin Core Python surface
NDNSF-DistributedInference/            # all DI policy/lifecycle/runtime
NDNSF-DistributedRepo/                 # canonical Repo contract/runtime
NDNSF-UAV-APP/                         # UAV domain behavior
tests/                                 # contract and regression gates
Experiments/                           # matched MiniNDN validation
```

**Structure Decision**: Preserve the existing multi-project repository. Spec
084 remains governance-only; child specs move code across current ownership
boundaries and do not create a shared framework for application policy.

## Complexity Tracking

| Temporary complexity | Why needed | Exit condition |
|---|---|---|
| Mixed typed/legacy compatibility epoch | Rolling migration | All producers and consumers typed; conflict counter remains zero |
| Two Repo runtime candidates before ADR | Preserve current evidence while deciding | ADR selects one; parity and persistence migration pass |
| Experimental advisory coordinator package | Preserve research comparison | Evidence justifies retention, otherwise delete |

Post-design constitution check remains PASS. Production implementation is
blocked until its child spec passes audit. All temporary complexity has an
explicit owner and exit condition.
