# Tasks: NDNSF Occam Simplification Program

**Input**: `spec.md`, `plan.md`, `research.md`, `data-model.md`, and
`contracts/` under `specs/084-ndnsf-occam-simplification/`

**Execution rule**: Spec 084 is an umbrella. These tasks create gates, child
features, audits, and acceptance evidence. Production edits are executed only
from the audited child feature named by `contracts/child-feature-map.md`.

**Tests**: Required. Every removal is test-first, evidence-gated, independently
revertible, and subject to `contracts/experiment-gates.md` when measured.

## Phase 1: Freeze Program Baseline And Decisions

**Purpose**: Establish a truthful baseline before any child modifies production
code. Preserve all unrelated dirty work.

- [x] T001 Record `git status --short`, branch, HEAD, dirty-file owner, excluded paths, and rollback boundary in `evidence/program-baseline.md`.
- [x] T002 Run `codegraph status .` and save exact symbol/caller queries for every mechanism in `contracts/ownership-matrix.md` to `evidence/caller-inventory.md`.
- [x] T003 Record the current DI lease reality in `evidence/di-lease-baseline.md`, including the coordinator/refCount authority and the missing `ndnsf.runtime_telemetry.ExecutionLease` import target in `pythonWrapper/ndnsf/service.py`.
- [x] T004 [P] Record exact current Core build, unit, security, normal, Targeted, collaboration, and bootstrap commands/results in `evidence/core-baseline.md`.
- [x] T005 [P] Record exact current DI unit, NativeTracer, Qwen, GUI/headless, and multi-user commands/results in `evidence/di-baseline.md`.
- [x] T006 [P] Record exact current Repo exact-packet, cache, restart, HA, Targeted, repair, and catalog commands/results in `evidence/repo-baseline.md`.
- [x] T007 [P] Record exact current UAV protocol, authority, mission, stream, FEC, and MiniNDN commands/results in `evidence/uav-baseline.md`.
- [x] T008 Replace every relevant `DISCOVER` entry in `contracts/regression-matrix.md` with an exact canonical command or an explicit owner/blocker in `evidence/regression-command-index.md`.
- [x] T009 Freeze module-specific thresholds, topology, load, seed set, warmup, repetitions, metrics, and canonical result paths under `contracts/experiment-gates.md` in `evidence/performance-thresholds.md` before treatment runs.
- [x] T010 Create one BLOCKED removal-gate record per ownership-matrix mechanism under `evidence/removal-gates/`, including its change class and mandatory fields from `contracts/removal-gate.md`.
- [x] T011 Implement and test a read-only forbidden-symbol/caller audit in `tools/maintenance/ndnsf_occam_audit.py` and `tests/python/test_ndnsf_occam_audit.py`; it must distinguish active code, tests, generated code, docs, and historical specs.
- [x] T012 Validate Spec Kit prerequisites, GSD health, CodeGraph freshness, and evidence completeness; record exact output in `evidence/workflow-gates.md`.

**Checkpoint**: Baselines and thresholds are frozen; no production code has
changed under Spec 084.

---

## Phase 2: Child 085 - Core Boundary And Fail-Closed Leases (US1)

**Goal**: Give every exclusive resource a real provider authority and remove
application policy from the Core surface without relying on a global coordinator.

- [x] T013 [US1] Create child feature 085 through Spec Kit without overwriting an existing feature; its spec MUST incorporate `contracts/di-lease-authority.md` and list exact current source symbols and target modules.
- [x] T014 [US1] Require child 085 tests for missing lease type/import, authority timeout, duplicate prepare/commit, conflicting digest, partial prepare/commit, abort, renewal, release loss, TTL expiry, stale epoch, provider restart, and eviction during execution.
- [x] T015 [US1] Require child 085 to define the DI-owned descriptive deployment record separately from provider-authoritative lease state and to prohibit global refCount as an execution authority.
- [x] T016 [US1] Require child 085 to inventory and relocate DI deployment/materializer/retry and Repo producer symbols currently exposed by `pythonWrapper/ndnsf/`, while retaining generic Core large-data, status, telemetry, rejection, and lease primitives.
- [x] T017 [US1] Run `speckit-audit` pre-implementation on child 085 and resolve every blocking finding before implementation begins; save the report in the child evidence directory.
- [x] T018 [US1] Accept child 085 only after Core/security regressions and coordinator-off multi-user MiniNDN prove zero synthetic leases, zero conflicting committed roles, bounded abort/replan, and independent rollback; link evidence in `evidence/child-085-acceptance.md`.

**Checkpoint**: Authority loss fails closed; Core contains no DI/Repo policy.

---

## Phase 3: Child 086 - V2 Invocation And Permission Migration (US2)

**Goal**: Remove V1 without deleting the provider/service authorization data
used by current V2 security paths.

- [x] T019 [US2] Create child feature 086 with a symbol-level V1 inventory and `contracts/permission-v2-migration.md`; identify exact declarations, definitions, registrations, callers, tests, docs, and build entries.
- [x] T020 [US2] Freeze an external ABI/release decision for `PublishRequest`, split ServiceName/FunctionName APIs, Bloom-filter naming, legacy handlers, Direct aliases, and permission callbacks in the child ADR.
- [x] T021 [US2] Require child 086 to introduce and test a unified service-authorization record containing providerServiceName, serviceName, permissionKind, and policyEpoch before changing `UserPermissionTable`.
- [x] T022 [US2] Require current encrypted PermissionResponse, user authorization, provider role authorization, NAC-ABE routing, one-time token, replay, bootstrap, normal, Targeted, and collaboration tests before and after migration.
- [x] T023 [US2] Require individual zero-caller proof before deleting `searchByFunctionName`, token-name decode callbacks/utilities, legacy parsers, handler overloads, or BloomFilter build targets; broad wildcard deletion is prohibited.
- [x] T024 [US2] Require child tasks to name exact repository paths and symbols rather than `ServiceUser.*`, `planner_registry.py`, or wildcard test placeholders.
- [x] T025 [US2] Run `speckit-audit` pre-implementation on child 086 and resolve every blocking finding.
- [ ] T026 [US2] Accept child 086 only after full C++/Python builds, security regressions, forbidden-symbol scans, and matched normal/Targeted MiniNDN pass the frozen thresholds; link evidence in `evidence/child-086-acceptance.md`.

**Checkpoint**: V2 provider/service authorization remains functional and no
active V1 invocation implementation remains.

---

## Phase 4: Child 087 - DI Policy And Lifecycle Ownership (US3)

**Goal**: Default DI is pure user-side planning plus provider admission;
experimental advice/cache cannot authorize or silently affect execution.

- [x] T027 [US3] Create child feature 087 after child 085 acceptance; include exact paths under `NDNSF-DistributedInference/ndnsf_distributed_inference/` and explicitly created target modules where no current file exists.
- [x] T028 [US3] Require coordinator-off multi-user correctness, deployment publication without authority semantics, executable-only default planner registry, and default-off semantic-cache boundary tests.
- [x] T029 [US3] Require advisory coordination to move under `experimental/advisory_coordination/`, remain non-authoritative/default-off, and expose no Core or default DI imports.
- [x] T030 [US3] Require semantic cache to move under `experimental/semantic_cache/`; retain provider-local Exact Forward Cache as a distinct exact-match optimization.
- [x] T031 [US3] Require retry policy to stay application-owned and to accept explicit idempotency metadata rather than infer safety from error strings.
- [x] T032 [US3] Freeze the advisory-retention experiment before running it: select one primary metric; require at least ten matched runs, a practical effect of at least 10%, a paired 95% bootstrap confidence interval excluding zero, no completion/latency threshold violation, and report conflict rate, completion, p50/p95, stable RPS, and added hop cost.
- [x] T033 [US3] Run `speckit-audit` pre-implementation on child 087 and resolve every blocking finding.
- [ ] T034 [US3] Accept child 087 only after NativeTracer/Qwen unit and MiniNDN workflows succeed coordinator-off; retain advisory code only if the predeclared T032 statistical and practical-effect gate passes, otherwise delete it; link `evidence/child-087-acceptance.md`.

**Checkpoint**: DI correctness has no coordinator dependency and experimental
features are visibly optional.

---

## Phase 5: Child 088 - Repo Canonical Contract And Runtime (US4)

**Goal**: Select, then converge on, one Repo contract and authoritative runtime
without losing exact packets, persistence, HA, or repair.

- [x] T035 [US4] Create child feature 088 with `contracts/repo-decision-gate.md`; leave canonical language/runtime and shared-versus-operation-specific naming explicitly UNDECIDED at feature creation.
- [x] T036 [P] [US4] Freeze black-box fixtures consumable by both candidates for exact signed packets, SQLite schema/restart, cache, manifest, catalog, quorum, tombstone, idempotency, failover, repair, Targeted, malformed input, and metrics.
- [x] T037 [P] [US4] Inventory current C++ and Python public/internal operations, names, schemas, authorization, discovery, persistence ownership, and parity gaps in the child evidence directory.
- [x] T038 [US4] Produce and approve a Repo ADR evaluating semantic parity, security, public/internal boundaries, persistence migration, crash recovery, concurrency/backpressure, observability, maintainability, operational complexity, and matched performance; one timing comparison is insufficient.
- [x] T039 [US4] Freeze the canonical public object API and versioned private replication/repair protocol only after T038; define an enforceable internal authorization namespace/policy and ordinary-client negative tests.
- [x] T040 [US4] Require child implementation tasks to migrate parity in independently revertible slices: exact storage/restart, bounded hot cache, quorum/tombstone, catalog/anti-entropy, read failover, repair, binding/client, then duplicate deletion.
- [x] T041 [US4] Require raw-payload helpers to segment/sign client-side and call canonical exact-Data insertion; they must not create a second provider storage model.
- [x] T042 [US4] Require stored-state upgrade, restart, rollback-open/downgrade decision, exact wire-byte, Targeted capability epoch, and internal-operation authorization tests before deleting either candidate.
- [x] T043 [US4] Run `speckit-audit` pre-implementation on child 088 after the ADR and resolve every blocking finding.
- [x] T044 [US4] Accept child 088 only after at least three matched 60-second campaigns each retain 30/30, required W, zero invalid finalized replicas, frozen repair coverage, and performance thresholds; link `evidence/child-088-acceptance.md`.

**Checkpoint**: Repo has one selected source of truth and ordinary clients
cannot invoke internal HA mutation operations.

---

## Phase 6: Child 089 - Core Stream Parity And UAV Migration (US5)

**Goal**: One generic C++ stream state engine, with UAV codec and safety policy
remaining in the UAV application.

- [x] T045 [US5] Create child feature 089 using `contracts/stream-parity.md`; name exact Core C++, pybind/Python, UAV consumer, test, and experiment paths.
- [x] T046 [P] [US5] Freeze parity fixtures for session identity, reorder, duplicate, gap, skip/deadline input, stale session, pending count/bytes, overflow, metrics, malformed data, unknown version, and adaptive state.
- [x] T047 [P] [US5] Define C++ ownership/thread-safety/callback guarantees and TLV-to-Python field/default/error conversion before binding implementation.
- [x] T048 [US5] Require the child to migrate only generic sequence/reorder/gap/health/adaptive behavior; H264 framing, FEC codec, ROI, MAVLink, mission, preflight, authority, decoder backlog policy, and labels remain UAV-owned.
- [x] T049 [US5] Require a forbidden-use test proving static files, models, catalog snapshots, and planned tensor bundles use exact-name segmented retrieval rather than StreamChunk.
- [x] T050 [US5] Run `speckit-audit` pre-implementation on child 089 and resolve every blocking finding.
- [ ] T051 [US5] Accept child 089 only after C++/Python parity tests and at least three matched UAV MiniNDN loss campaigns preserve stale rejection, FEC recovery, bounded buffering, gaps/drops, completion, and latency thresholds; link `evidence/child-089-acceptance.md`.

**Checkpoint**: One generic stream algorithm remains; UAV domain behavior and
static-object semantics are unchanged.

---

## Phase 7: Child 090 - Typed Envelopes Without Semantic Loss (US6)

**Goal**: Remove compatibility aliases after a bounded epoch without deleting
domain state that merely shares a field name with a generic envelope.

- [ ] T052 [US6] Create child feature 090 only after children 086-089 stabilize their contracts; use the `FieldDisposition` model from `data-model.md`.
- [ ] T053 [US6] Inventory every capability, runtime, operation status, rejection, network, lease, Repo, DI, and UAV field as legacy-alias, domain-state, transport-metadata, or unknown; unknown fields block removal.
- [ ] T054 [US6] Define schema versions, typed-authority conflict rules, malformed/unknown behavior, legacy/conflict counters, stored-state migration, compatibility deadline, and exit criteria.
- [ ] T055 [P] [US6] Require typed-only, legacy-only, matching dual, conflicting dual, unknown-version, malformed, rolling-upgrade, restart, and rollback fixtures for Core, DI, Repo, and UAV.
- [ ] T056 [US6] Remove only fields classified as legacy aliases after current producers emit typed-only and mixed-version counters remain within the frozen exit criteria; retain application-domain status such as residency.
- [ ] T057 [US6] Run `speckit-audit` pre-implementation on child 090 and resolve every blocking finding.
- [ ] T058 [US6] Accept child 090 only after mixed-version and typed-only MiniNDN smoke, full regressions, zero forbidden legacy emission, zero unexplained conflicts, stored-state migration, and independent rollback; link `evidence/child-090-acceptance.md`.

**Checkpoint**: One representation exists per contract, with no semantic state
lost merely to reduce field count.

---

## Phase 8: Conditional Structural Cleanup And Final Acceptance

**Purpose**: Remove proven dead structure after behavior converges. File splits
are conditional maintainability work, not automatic Occam success criteria.

- [ ] T059 Re-run CodeGraph and source metrics after child acceptance; decide separately whether `service.py`, `ServiceUser/ServiceProvider`, or `GroundStationServiceContainer` still require splitting, and create dedicated refactor specs rather than mixing moves with deletion.
- [ ] T060 Remove dead includes, build targets, compatibility tests, and obsolete examples found by the audit only after their individual removal gates are READY; archive historical specs instead of deleting evidence by string scan.
- [ ] T061 Map every FR and SC to its child requirement, task, exact command, and evidence path in `traceability.md`; no blanket “final phase verifies all” mapping is allowed.
- [ ] T062 [P] Run final Core build/security/normal/Targeted/collaboration acceptance and record exact commands/results in `evidence/final-core.md`.
- [ ] T063 [P] Run final DI coordinator-off NativeTracer/Qwen/multi-user acceptance and record exact commands/results in `evidence/final-di.md`.
- [ ] T064 [P] Run final Repo exact-packet/persistence/HA/Targeted/recovery/repair/catalog acceptance and record exact commands/results in `evidence/final-repo.md`.
- [ ] T065 [P] Run final UAV protocol/mission/authority/stream/video acceptance and record exact commands/results in `evidence/final-uav.md`.
- [ ] T066 Compare baseline/final public symbols, source count, schema fields, maintenance paths, completion, p50/p95, failure reasons, and resource use in `evidence/final-occam-report.md` without claiming improvement where measurements disagree.
- [ ] T067 Run `speckit-analyze`, `speckit-converge`, GSD verification, CodeGraph boundary audit, and ARS adversarial review; append unresolved work to the owning child rather than declaring success.
- [ ] T068 Update `docs/ndnsf-core-app-boundary.md`, architecture indexes, English/Chinese module docs, and active agent context to the accepted ownership model.
- [ ] T069 Mark Spec 084 complete only when every child has PASS acceptance, every removal gate is READY or explicitly deferred with owner/reason/expiry, and every SC has reproducible evidence.

## Dependency Order

```text
Phase 1 baseline
  -> 085 Core/lease authority
      -> 086 V2 permission migration
      -> 087 DI policy isolation
  -> 088 Repo ADR and convergence
  -> 089 stream parity
  086 + 087 + 088 + 089 -> 090 typed migration
  all children -> final acceptance
```

- Repo fixture freezing may run in parallel with 085, but Repo implementation
  cannot begin before the ADR.
- Stream fixture freezing may run in parallel with Repo work.
- Typed migration is last because earlier children may change canonical fields.
- Structural refactors are separate specs when still justified after deletion.
- No task may stage unrelated dirty files or alter proposal-defense slides,
  NDN-SVS, secrets, certificates, or local identity state.
