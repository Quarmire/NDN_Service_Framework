# Feature Specification: NDNSF Occam Simplification

**Feature Branch**: `084-ndnsf-occam-simplification`

**Created**: 2026-07-10

**Status**: Complete - All Child Specifications Accepted

**Input**: Consolidate the NDNSF Core, DistributedInference, DistributedRepo,
and UAV implementations around one canonical contract per concern. Remove dead
or transitional mechanisms only after callers migrate and security,
distributed-correctness, and measured performance invariants remain intact.

Spec 084 governs scope, ownership, decision gates, validation, and completion
of the simplification program. Production implementation is performed only by
the child features listed in `contracts/child-feature-map.md`; 084 is not a
single cross-project rewrite authorization.

## User Scenarios & Testing

### User Story 1 - Correctness-Preserving Core Boundary (Priority: P1)

An NDNSF application developer can use the generic Core without importing DI,
Repo, UAV, model deployment, or advisory scheduling concepts. When a required
distributed lease authority is unavailable, the request fails or defers instead
of silently creating an untracked local substitute.

**Why this priority**: An unsafe fallback or an application-specific Core API
can invalidate all later simplification work.

**Independent Test**: Run Core Python/C++ contract and security regressions,
then verify that Core exports only service-neutral invocation, security,
large-data, stream, telemetry, status, and lease primitives.

**Acceptance Scenarios**:

1. **Given** the DI deployment authority is unavailable, **when** a user asks
   for an execution lease, **then** the operation returns a typed
   unavailable/rejected result and never returns an untracked local lease.
2. **Given** a non-DI application imports `ndnsf`, **when** its public API is
   inspected, **then** DI deployment, model artifact, advisory coordinator, and
   Repo producer classes are absent from the generic surface.
3. **Given** an existing DI deployment workflow, **when** it is migrated to a
   DI-owned manager, **then** its behavior remains available without adding a
   DI dependency to Core.

---

### User Story 2 - One Canonical Invocation Protocol (Priority: P1)

Application developers use only unified V2 `serviceName` request, ACK,
selection, response, and Targeted APIs. Split `ServiceName + FunctionName`,
Bloom-filter request names, deprecated permission tokens, and unused legacy
handler overloads no longer increase runtime or maintenance complexity.

**Why this priority**: The duplicate protocol path affects the two largest Core
translation units and every security-sensitive message parser.

**Independent Test**: Prove no repository caller uses the V1 symbols, remove
them, build all C++/Python targets, and run the normal, Targeted, NAC-ABE,
token/replay, bootstrap, negative-ACK, and collaboration regressions.

**Acceptance Scenarios**:

1. **Given** a normal or Targeted request, **when** names are generated and
   parsed, **then** only V2 unified-service helpers are used.
2. **Given** the built repository, **when** obsolete symbol and string scans run,
   **then** no active Bloom-filter invocation, split function-name API, Direct
   terminology, or deprecated service token remains.
3. **Given** an external compatibility concern, **when** the removal gate is
   evaluated, **then** removal stops unless a documented adapter or major-version
   boundary exists.

---

### User Story 3 - DI Owns DI Policy (Priority: P2)

NDNSF-DI owns deployment discovery, fragment maps, execution artifacts,
execution retry, semantic cache experiments, and optional multi-user advisory
planning. Core retains application-neutral mechanisms whose semantics and
enforcement belong in the framework. Two independent consumers are strong
promotion evidence, not an absolute prerequisite for a correctness primitive.

**Why this priority**: DI policy is currently spread across Core wrappers,
runtime modules, examples, and experiments.

**Independent Test**: Run NativeTracer/Qwen unit and MiniNDN workflows with the
DI-owned deployment manager while the advisory coordinator is disabled by
default and removable without changing correctness.

**Acceptance Scenarios**:

1. **Given** multiple users compete for providers, **when** advisory coordination
   is disabled, **then** provider admission leases and plan prepare/commit/abort
   prevent conflicting execution and bounded replanning remains possible.
2. **Given** advisory coordination is enabled as an experiment, **when** it is
   unavailable or stale, **then** users fall back to pure user-side planning
   without bypassing provider admission.
3. **Given** the default planner registry, **when** it is listed, **then** every
   advertised backend has an executable handler.
4. **Given** semantic caching is not explicitly enabled, **when** an LLM request
   runs, **then** approximate cached answers cannot affect selection or output.
5. **Given** a provider restarts or a multi-provider commit is partial, **when**
   the plan is evaluated, **then** stale lease epochs are rejected and no role
   executes until every selected provider has committed the same plan digest.

---

### User Story 4 - One Canonical Repo Runtime and Minimal Public Contract (Priority: P2)

Repo users see one service contract and one authoritative storage/catalog/repair
implementation selected by an explicit ADR rather than assumed in this
umbrella. Convenience adapters do not create a second object model, and
internal replication operations are not presented as general application APIs.

**Why this priority**: C++ and Python currently implement different service
surfaces and overlapping storage/catalog behavior.

**Independent Test**: Run the existing exact-packet, tiered-cache, HA,
quorum-failure, recovery, parallel-repair, Targeted, and catalog-merge campaigns
through the selected canonical runtime.

**Acceptance Scenarios**:

1. **Given** raw payload or already-segmented Data, **when** an object is inserted,
   **then** both client conveniences converge on exact-name NDN Data storage.
2. **Given** an external Repo client, **when** it inspects supported operations,
   **then** it sees a small stable public API while packet pull/batch/finalization
   remain internal replication operations.
3. **Given** all deployed providers support Targeted and pull merge, **when** the
   compatibility removal gate passes, **then** Normal-only and old catalog
   fallbacks are removed without reducing quorum completion or recovery.
4. **Given** an ordinary Repo client, **when** it attempts an internal
   replication or repair operation, **then** authorization rejects it and the
   internal operation is not advertised as a public service.

---

### User Story 5 - One Stream State Engine (Priority: P3)

Continuous UAV video, telemetry, and other live feeds share Core stream sequence,
reorder, gap, health, and adaptive-fetch state. UAV keeps codec, frame, FEC, ROI,
MAVLink, mission, and operator-safety policy. Static files and model objects
continue to use exact-name segmented retrieval rather than streams.

**Why this priority**: The current Core C++, Core Python, and UAV decoder paths
duplicate sequence and reorder behavior.

**Independent Test**: Run Core stream tests and UAV stream MiniNDN loss tests,
including stale-session rejection, duplicates, gaps, reorder, FEC recovery, and
bounded decoder backlog.

**Acceptance Scenarios**:

1. **Given** out-of-order live chunks, **when** the UAV consumer receives them,
   **then** Core reorder state determines emission and missing sequences.
2. **Given** a large static object, **when** it is transferred, **then** it uses
   exact names and SegmentFetcher-style retrieval, not StreamChunk wrapping.
3. **Given** Python stream users, **when** they use stream state, **then** they
   call bindings over the canonical C++ implementation instead of a parallel
   Python algorithm.

---

### User Story 6 - Typed Contracts Without Permanent Dual Encoding (Priority: P3)

Core, DI, Repo, and UAV exchange typed capability, runtime, status, rejection,
network, and lease envelopes without also maintaining indefinitely duplicated
flat legacy fields and string parsers.

**Why this priority**: Dual encoding doubles producer, consumer, test, and
documentation paths and makes contradictory values possible.

**Independent Test**: Run schema-version contract tests in mixed-version mode,
then typed-only mode; verify producers no longer emit legacy fields after the
migration epoch.

**Acceptance Scenarios**:

1. **Given** typed and legacy values disagree during migration, **when** a current
   consumer parses the ACK, **then** the typed value is authoritative and the
   conflict is counted.
2. **Given** the migration epoch has ended, **when** current producers and
   consumers communicate, **then** no legacy flat status/capability fields are
   emitted or required.
3. **Given** a domain state such as `DISK_RESIDENT`, **when** typed fields are
   inventoried, **then** it remains a domain field unless it is proven to be an
   alias of generic operation status.

### Edge Cases

- An external consumer outside this repository may still use a legacy ABI.
- A rolling deployment may temporarily contain typed-only and legacy-only nodes.
- Coordinator removal must not remove provider-local conflict prevention.
- Repo migration must preserve SQLite data and exact packet wire bytes.
- A Targeted token bootstrap failure must not be mistaken for provider failure.
- Stream consolidation must preserve UAV FEC frame recovery and skip policy.
- Removing retries must not strand idempotent DI operations that explicitly opt in.
- Existing dirty-worktree changes must be preserved and separated from cleanup commits.
- A provider restart invalidates leases from its prior lease epoch.
- A partial multi-provider commit must never become executable.
- Repo canonical-runtime selection must remain undecided until its ADR gate.
- Internal Repo operations must reject ordinary client identities.

## Requirements

### Functional Requirements

- **FR-001**: The project MUST publish a machine-checkable ownership matrix for
  every mechanism moved, removed, retained, or made experimental.
- **FR-002**: Security, permission, NAC-ABE, certificate bootstrap, one-time
  token, replay protection, and provider authorization behavior MUST remain
  unchanged unless a separate security specification approves a change.
- **FR-003**: Provider admission leases, execution leases, TTL, rejection
  reasons, prepare/commit/abort, lease epochs, idempotency, and rollback MUST
  remain the correctness path for exclusive distributed resources as defined
  by `contracts/di-lease-authority.md`.
- **FR-004**: The broken/untracked local execution-lease fallback MUST be
  captured by a regression and removed before advisory coordinator retirement;
  authority loss MUST fail closed.
- **FR-005**: Core MUST not expose DI deployment, model artifact, semantic cache,
  Repo data-plane producer, UAV, or application retry policy APIs.
- **FR-006**: DI deployment and lifecycle APIs MUST move behind a DI-owned
  manager that uses generic Core service, discovery, status, telemetry, and
  lease APIs.
- **FR-007**: Advisory coordination MUST be optional, disabled by default, and
  unable to authorize execution or bypass provider admission.
- **FR-008**: Legacy V1 invocation removal MUST follow
  `contracts/permission-v2-migration.md`: preserve current V2 provider/service
  authorization while removing split names, Bloom-filter request naming,
  deprecated token indexing, verified-dead parsers/callbacks, build references,
  tests, and documentation.
- **FR-009**: Targeted invocation MUST remain the known-provider low-latency API;
  Direct compatibility terminology MUST not remain in public interfaces.
- **FR-010**: The Repo project MUST select one canonical service contract and
  one authoritative implementation for storage, catalog, and repair through
  `contracts/repo-decision-gate.md`; Spec 084 MUST NOT preselect the language.
- **FR-011**: Repo public operations MUST be separated from private replication,
  repair, and anti-entropy operations by an enforceable naming/authorization
  boundary with negative tests.
- **FR-012**: Raw-payload Repo helpers MUST adapt to the exact-name Data model and
  MUST NOT create an independent object-storage model.
- **FR-013**: Core C++ MUST be the canonical stream state implementation; Python
  and UAV generic sequence/reorder/adaptive logic MUST migrate to it only after
  the behavior and binding contract in `contracts/stream-parity.md` passes.
- **FR-014**: UAV MUST retain application-specific H264, FEC codec, ROI,
  MAVLink, mission, preflight, and operator authority policy.
- **FR-015**: Static files, models, catalog snapshots, and planned tensor bundles
  MUST continue to use exact-name segmented large-data retrieval.
- **FR-016**: Typed envelopes MUST carry explicit schema versions and a bounded
  compatibility epoch before legacy aliases are removed; application-domain
  state MUST not be deleted merely because it is named `status`.
- **FR-017**: Planner backends without executable handlers MUST not be registered
  in the default planner registry.
- **FR-018**: Semantic cache functionality MUST be experimental, opt-in, and
  excluded from default provider selection; Exact Forward Cache remains a
  separate exact-match optimization.
- **FR-019**: Automatic application-level retry MUST require explicit
  idempotency and remain application-owned; generic Core invocation MUST not
  infer retry safety from error strings.
- **FR-020**: Each removal phase MUST have a pre-removal caller scan, focused
  contract tests, module regressions, and a MiniNDN gate where network behavior
  changes.
- **FR-021**: Performance-sensitive migrations MUST compare matched before/after
  runs under `contracts/experiment-gates.md` and reject changes that materially
  regress established canonical results without a predeclared tradeoff.
- **FR-022**: Every phase MUST be independently revertible and MUST not combine
  unrelated dirty-worktree changes.
- **FR-023**: Spec 084 MUST remain an umbrella; production edits MUST be owned by
  audited child features with independent rollback and evidence.

### Key Entities

- **OwnershipDecision**: Mechanism, current owner, target owner, disposition,
  rationale, dependencies, removal gate, and verification evidence.
- **CompatibilityEpoch**: Schema or API, producer version, consumer version,
  start/end conditions, conflict rule, telemetry, and removal status.
- **RemovalGate**: Caller scan, adapter decision, tests, MiniNDN campaign,
  performance threshold, rollback command, and approval state.
- **CanonicalRuntimeDecision**: Concern, selected implementation, migration
  source, retained adapters, persistence compatibility, and completion evidence.
- **RegressionMatrix**: Invariant, affected modules, command, expected result,
  evidence path, and blocking severity.
- **ProviderLeaseAuthority**: Provider, lease epoch, plan digest, role set,
  prepare/commit state, expiry, and idempotency key.

## Success Criteria

### Measurable Outcomes

- **SC-001**: Core public API contains zero DI deployment, semantic cache, UAV,
  Repo-specific producer, or application retry symbols.
- **SC-002**: Repository scans find zero active V1 split-name/Bloom-filter
  invocation call sites and zero default planner backends without handlers.
- **SC-003**: No execution path returns a lease that is not tracked by its
  authoritative lease owner.
- **SC-004**: Repo has one documented canonical wire contract and one
  authoritative storage/catalog/repair implementation.
- **SC-005**: Current Repo HA campaigns retain 30/30 completion, required write
  quorum, zero invalid finalized replicas, and required repair coverage.
- **SC-006**: Core security, normal/Targeted invocation, bootstrap, negative ACK,
  collaboration, DI, UAV, and Repo focused regressions all pass.
- **SC-007**: UAV stream loss tests preserve stale-session rejection, FEC
  recovery, and bounded buffering after using the Core reorder implementation.
- **SC-008**: Typed-only producer/consumer tests pass with no legacy fields;
  mixed-version tests pass only during the documented compatibility epoch.
- **SC-009**: Matched MiniNDN campaigns satisfy the frozen repetitions,
  completion, p50/p95, and evidence rules in `contracts/experiment-gates.md`;
  unexplained regressions block deletion.
- **SC-010**: Every removed mechanism has a recorded caller scan, replacement or
  no-replacement decision, verification evidence, and rollback point.
- **SC-011**: Every child feature in `contracts/child-feature-map.md` has a PASS
  audit, completed acceptance evidence, and an independent rollback point before
  Spec 084 is marked complete.

## Assumptions

- The repository is the authoritative caller set for immediate source cleanup;
  external ABI compatibility requires an explicit release decision.
- Current V2 unified-service APIs and Targeted terminology are canonical.
- Current MiniNDN canonical results are baselines, not promises that every
  cleanup improves performance.
- Python may remain an orchestration/client layer. The Repo ADR determines the
  authoritative runtime; storage, catalog, and repair must eventually have only
  one source of truth.
- Advisory scheduling may remain as research code after it leaves the default
  runtime and Core public surface.
- This feature does not modify proposal-defense slides or NDN-SVS.
