# Tasks: DI Runtime-Aware User-Side Planner

**Input**: Design documents from `specs/047-di-runtime-aware-user-planner/`

**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

**Tests**: Include tests because this feature changes planner, ACK/selection semantics, lease validation, and MiniNDN evaluation behavior.

**Organization**: Tasks are grouped by independently testable user stories.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Prepare fixtures and shared schema locations without changing runtime behavior.

- [X] T001 Create runtime-aware planner fixture directory in `examples/python/NDNSF-DistributedInference/native_di_tracer/runtime_aware_fixtures/`
- [X] T002 [P] Add sample model fragment inventory fixture in `examples/python/NDNSF-DistributedInference/native_di_tracer/runtime_aware_fixtures/provider_fragments.json`
- [X] T003 [P] Add sample directed provider network matrix fixture in `examples/python/NDNSF-DistributedInference/native_di_tracer/runtime_aware_fixtures/provider_network_matrix.json`
- [X] T004 [P] Add sample multi-user workload fixture in `examples/python/NDNSF-DistributedInference/native_di_tracer/runtime_aware_fixtures/multi_user_requests.json`
- [X] T005 Add feature flag/profile knob for runtime-aware user-side planning in `examples/di-native-tracer.runtime.json`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Define NDNSF core reusable envelopes first, then DI-specific schemas built on top of them.

- [X] T006 Define generic `GenericAckMetadata`, `GenericProviderRuntimeHint`, `PeerNetworkMetric`, `GenericAdmissionLease`, and `GenericLeaseValidationResult` schema helpers in NDNSF core message/runtime support files under `ndn-service-framework/`
- [X] T007 [P] Add NDNSF core serialization/deserialization tests for generic ACK metadata, admission lease, and peer telemetry envelopes in `tests/python/test_ndnsf_core_admission_metadata.py` or an equivalent focused regression
- [X] T008 Add DI-specific `ModelFragmentKey`, `DiFragmentRuntimeState`, `DiProviderRuntimeState`, `DiLeaseResourceBinding`, and `ProviderNetworkMatrix` schemas in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- [X] T009 [P] Add DI serialization/deserialization unit tests for fragment, residency, DI lease binding, and network matrix schemas in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [X] T010 Add `PlanTemplate` versus `RuntimeAssignment` schema separation in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- [X] T011 [P] Add fixture loader helpers for fragment inventory and network matrix in `examples/python/NDNSF-DistributedInference/native_di_tracer/runtime_aware_fixtures/loader.py`
- [X] T012 Document core-versus-DI runtime-aware planner data assumptions in `docs/NDNSF-DI-runtime-workflow.md`
- [X] T013 Run `python3 -m py_compile` for changed Python modules plus generic core and DI schema tests

**Checkpoint**: Schemas and fixtures exist; no service behavior changes yet.

---

## Phase 3: User Story 1 - Runtime-aware assignment from provider ACKs (Priority: P1) MVP

**Goal**: User-side planner selects runtime assignments from provider ACK state and fragment residency.

**Independent Test**: Deterministic fixture proves GPU-loaded, CPU-resident, disk-resident, repo-available, and missing fragments are scored in the expected order.

### Tests for User Story 1

- [X] T014 [P] [US1] Add planner fixture test for fragment residency ordering in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [X] T015 [P] [US1] Add planner fixture test excluding providers without valid runtime state when runtime-aware mode is required in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [X] T016 [P] [US1] Add planner fixture test for conservative fallback when runtime-aware mode is optional in `tests/python/test_ndnsf_di_runtime_aware_planner.py`

### Implementation for User Story 1

- [X] T017 [US1] Implement DI residency cost constants and score breakdown structure in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- [X] T018 [US1] Implement runtime-aware candidate scoring helper that consumes generic core hints plus DI payloads in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- [x] T019 [US1] Wire runtime-aware scoring into the NativeTracer user-side planner path in `examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py`
- [x] T020 [US1] Emit selected assignment and rejected candidate reasons in NativeTracer planner output in `examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py`
- [X] T021 [US1] Validate US1 with `python3 tests/python/test_ndnsf_di_runtime_aware_planner.py`

**Checkpoint**: User-side planner can choose assignments from runtime state without lease enforcement.

---

## Phase 4: User Story 2 - Multi-user conflict control through provider leases (Priority: P1)

**Goal**: Providers issue and validate short-lived leases so concurrent users cannot all consume the same stale resource state.

**Independent Test**: Two synthetic users race for one single-slot provider role; only valid lease consumption triggers execution.

### Tests for User Story 2

- [X] T022 [P] [US2] Add generic lease manager unit test for grant, expire, consume, and release behavior in `tests/python/test_ndnsf_core_admission_metadata.py`
- [X] T023 [P] [US2] Add generic selection validation test for expired/mismatched/already-consumed leases in `tests/python/test_ndnsf_core_admission_metadata.py`
- [X] T024 [P] [US2] Add DI binding validation test for role/fragment mismatch in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [x] T025 [P] [US2] Add C++ or shell smoke test ensuring provider rejects invalid lease before role execution in a focused NDNSF runtime regression
- [x] T026 [P] [US2] Add compatibility smoke test proving a non-lease service still executes with the current ACK/Selection/Response path

### Implementation for User Story 2

- [x] T027 [US2] Implement generic provider lease table model in NDNSF core runtime support under `ndn-service-framework/`
- [x] T028 [US2] Extend ACK runtime payload builder with optional generic `GenericAdmissionLease` fields in `ndn-service-framework/ServiceProvider.cpp`
- [x] T029 [US2] Extend selection payload parsing to optionally carry generic lease id, service name, and resource binding proof in `ndn-service-framework/ServiceUser.cpp`
- [x] T030 [US2] Implement opt-in provider-side generic lease validation before selected service execution in `ndn-service-framework/ServiceProvider.cpp`
- [x] T031 [US2] Preserve current non-lease service behavior when lease validation is disabled in `ndn-service-framework/ServiceProvider.cpp` and `ndn-service-framework/ServiceUser.cpp`
- [x] T032 [US2] Implement DI resource binding validation for role id and fragment key in the NDNSF-DI provider path
- [x] T033 [US2] Add structured generic and DI lease rejection reasons to provider/user diagnostics in `ndn-service-framework/ServiceProvider.cpp`, `ndn-service-framework/ServiceUser.cpp`, and DI runtime logs
- [x] T034 [US2] Validate US2 with core lease unit tests, DI binding tests, non-lease compatibility smoke test, and focused provider selection smoke test

**Checkpoint**: Concurrent users can receive independent ACKs, but provider lease validation controls execution.

---

## Phase 5: User Story 3 - Edge-aware placement using provider-to-provider network metrics (Priority: P1)

**Goal**: Planner scores dependency edges using directed provider-to-provider bandwidth, RTT, loss, jitter, staleness, and confidence.

**Independent Test**: Fixture proves edge-aware scoring avoids a poor provider-pair dependency edge even when compute-only ranking prefers it.

### Tests for User Story 3

- [X] T035 [P] [US3] Add generic directed peer metric parsing test in `tests/python/test_ndnsf_core_admission_metadata.py`
- [X] T036 [P] [US3] Add DI edge-cost scoring test for bandwidth/RTT/loss/jitter in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [X] T037 [P] [US3] Add stale/unknown metric penalty test in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [X] T038 [P] [US3] Add graph assignment test where edge-aware placement beats compute-only placement in `tests/python/test_ndnsf_di_runtime_aware_planner.py`

### Implementation for User Story 3

- [X] T039 [US3] Implement generic `PeerNetworkMetric` envelope support in NDNSF core runtime metadata paths
- [X] T040 [US3] Implement DI `ProviderNetworkMatrix` helper over generic peer metrics in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- [X] T041 [US3] Implement DI dependency edge byte-size and transfer-cost estimator in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- [x] T042 [US3] Integrate node-cost plus edge-cost assignment scoring in `examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py`
- [x] T043 [US3] Add edge-cost details to selected assignment output in `examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py`
- [X] T044 [US3] Validate US3 with graph assignment fixture tests

**Checkpoint**: Planner treats NDNSF-DI as graph placement, not independent provider ranking.

---

## Phase 6: User Story 4 - Replan after stale runtime state (Priority: P2)

**Goal**: User-side planner performs bounded replan when selected leases/providers become invalid before or during execution.

**Independent Test**: Forced lease rejection triggers replan and produces a structured `ReplanRecord`.

### Tests for User Story 4

- [X] T045 [P] [US4] Add replan record serialization test in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [x] T046 [P] [US4] Add bounded replan test for `FRAGMENT_EVICTED` in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [x] T047 [P] [US4] Add max-attempt failure test with structured planner reason in `tests/python/test_ndnsf_di_runtime_aware_planner.py`

### Implementation for User Story 4

- [x] T048 [US4] Implement `ReplanRecord` model and exclusion list handling in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- [x] T049 [US4] Add bounded replan loop to NativeTracer user-side planner path in `examples/python/NDNSF-DistributedInference/native_di_tracer/user_driver.py`
- [x] T050 [US4] Emit replan count and reason into planner metrics in `examples/python/NDNSF-DistributedInference/native_di_tracer/user_driver.py`
- [x] T051 [US4] Validate US4 with fixture tests

**Checkpoint**: Runtime-aware user-side planning recovers from stale leases within bounded attempts.

---

## Phase 7: User Story 5 - Evidence through MiniNDN multi-user campaigns (Priority: P2)

**Goal**: MiniNDN campaigns compare static user-side planning against runtime-aware lease and edge-aware planning.

**Independent Test**: One documented MiniNDN command emits latency, lease, residency, edge-cost, replan, and utilization evidence.

### Tests for User Story 5

- [X] T052 [P] [US5] Add dry-run test for multi-user runtime-aware campaign arguments in `tests/python/test_ndnsf_di_runtime_aware_campaign.py`
- [X] T053 [P] [US5] Add parser test for planner metrics aggregation in `tests/python/test_ndnsf_di_runtime_aware_campaign.py`

### Implementation for User Story 5

- [X] T054 [US5] Add multi-user workload mode to `Experiments/NDNSF_DI_NativeTracer_Minindn.py`
- [X] T055 [US5] Add asymmetric provider-to-provider link profile to `Experiments/Topology/AI_Lab.conf` or a new DI topology fixture
- [X] T056 [US5] Add runtime-aware planner mode flags to `tools/ndnsf_runtime.py di run` passthrough documentation and profile defaults
- [X] T057 [US5] Emit planner metrics JSON/CSV from MiniNDN campaign in `Experiments/NDNSF_DI_NativeTracer_Minindn.py`
- [X] T058 [US5] Add campaign summary fields for p50/p95 latency, success rate, utilization, lease counters, residency counters, edge-cost summary, and replan count in `Experiments/NDNSF_DI_NativeTracer_Minindn.py`
- [X] T059 [US5] Run a short MiniNDN validation campaign and save canonical command/results summary under `docs/` or tracked experiment documentation

**Checkpoint**: Feature has measurable MiniNDN evidence for multi-user and provider-pair network behavior.

---

## Final Phase: Polish & Cross-Cutting Concerns

**Purpose**: Documentation, security checks, and regression cleanup.

- [X] T060 [P] Update `docs/NDNSF-DI-runtime-workflow.md` with runtime-aware planner workflow and metrics outputs
- [X] T061 [P] Update `docs/native-di-roadmap.md` with plan-template versus runtime-assignment and edge-aware planning rationale
- [X] T062 [P] Update generic NDNSF framework docs to describe reusable admission lease, ACK metadata, and peer telemetry boundaries
- [X] T063 Run focused token/security regressions covering UserToken/ProviderToken and selection validation paths
- [X] T064 Run generic core admission metadata tests
- [X] T065 Run `python3 tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [X] T066 Run `python3 tests/python/test_ndnsf_di_runtime_aware_campaign.py`
- [X] T067 Run `python3 tools/ndnsf_runtime.py di validate`
- [X] T068 Run `git diff --check`
- [X] T069 Record final validation commands and known limitations in `specs/047-di-runtime-aware-user-planner/quickstart.md`

## Follow-up: Real Provider Fragment Inventory Evidence

**Purpose**: Close the gap between fixture-reported residency and provider-local
runtime state so multi-user campaigns can prove model-fragment reuse.

- [X] T070 Add `ProviderFragmentInventoryManager` and `ProviderFragmentInventoryEntry` to `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- [X] T071 Add inventory unit coverage proving actual disk files become `DISK_RESIDENT`, runtime marks become `CPU_RESIDENT`/`GPU_LOADED`, and ACK metadata carries the resulting state
- [X] T072 Fix campaign metrics aggregation so `residencyCounters` count selected fragment residency instead of selected provider names
- [X] T073 Add campaign metrics aggregation for direct lease counters and `maxStableRps` from RPS sweep entries
- [X] T074 Document provider-local inventory and multi-user campaign evidence in `docs/NDNSF-DI-runtime-workflow.md` and `quickstart.md`
- [X] T075 Validate with runtime-v1 tests, runtime-aware campaign tests, py_compile, dry-run multi-user campaign, and diff check

## Follow-up: Native Provider Inventory Events and RPS Sweep

**Purpose**: Connect the C++ NativeProviderHandler artifact lifecycle to
campaign evidence and provide a repeatable multi-user RPS sweep wrapper.

- [X] T076 Emit `NDNSF_DI_FRAGMENT_INVENTORY` events from `NativeProviderHandler` when artifacts become disk-resident, CPU/GPU-loaded, observed during execution, and evicted at provider shutdown
- [X] T077 Aggregate provider fragment inventory events from MiniNDN provider logs into `summary.json` and `planner-metrics.json`
- [X] T078 Add focused campaign tests for inventory event parsing and runtime-aware RPS sweep dry-run command generation
- [X] T079 Add `Experiments/NDNSF_DI_RuntimeAware_RpsSweep.py` as the repeatable wrapper for runtime-aware multi-user RPS sweeps
- [X] T080 Document the provider inventory event bridge, RPS sweep command, and current NativeTracer lease limitation

## Follow-up: NativeTracer Generic Admission Lease Integration

**Purpose**: Turn the existing NDNSF core generic admission lease mechanism into
a real NativeTracer full-network path so multi-user campaigns can measure
resource conflict control instead of only inventory reuse.

- [X] T081 Commit the stable provider-inventory and RPS-sweep work before lease edits
- [X] T082 Add NativeTracer lease task plan and validation notes to `docs/NDNSF-DI-runtime-workflow.md` and `quickstart.md`
- [X] T083 Extend `SelectedParticipant` / `AckSelectionCandidate` handling so collaboration selection can carry lease id and resource binding from ACK payload into Selection assignment payload
- [X] T084 Make `DI_NativeProviderExecutable` grant one generic admission lease per successful readiness ACK, with role/fragment resource binding proof
- [X] T085 Enable `setGenericAdmissionLeaseRequired()` for NativeTracer services and preserve non-lease compatibility when the flag is disabled
- [X] T086 Add tests for lease payload extraction, assignment payload merging, and provider accept/reject log aggregation
- [X] T087 Run focused C++ build/unit tests plus Python campaign tests
- [X] T088 Run a short lease-enabled multi-user MiniNDN RPS sweep and record observed lease counters, residency counters, p50/p95, and max stable RPS

Validation evidence for T088:

```text
resultDir=/tmp/ndnsf-di-lease-smoke-final/rps-0p2
status=SUCCESS
requestCount=2
successCount=2
successRate=1.0
leaseCounters.granted=12
leaseCounters.consumed=8
leaseCounters.rejected=0
observedResidencyCounters.CPU_RESIDENT=12
observedResidencyCounters.DISK_RESIDENT=4
p50Ms=126.8783830000757
p95Ms=252.5117709992628
maxStableRps=0.2
providerFragmentInventory.eventCounters.EXECUTION_OBSERVED=8
```

Extended lease/no-lease sweep evidence from 2026-07-05:

```text
closedLoopLeaseDir=/tmp/ndnsf-di-lease-rps-sweep-enabled-20260705
closedLoopNoLeaseDir=/tmp/ndnsf-di-lease-rps-sweep-disabled-20260705
rps=0.2,0.4,0.8,1.2
requests=4
concurrency=2
leaseEnabledResult=all points SUCCESS, successRate=1.0, maxStableRps=1.2
noLeaseResult=all points SUCCESS, successRate=1.0, maxStableRps=1.2
leaseCounters.enabled=granted=16 consumed=16 rejected=0 at every point
leaseCounters.disabled=granted=0 consumed=0 rejected=0 at every point
observedThroughput=about 0.203 RPS at every target RPS
interpretation=closed-loop wrapper validates correctness but does not create higher offered load.
```

Open-loop comparison evidence from 2026-07-05:

```text
openLoopLeaseDir=/tmp/ndnsf-di-lease-rps-sweep-enabled-openloop-20260705
openLoopNoLeaseDir=/tmp/ndnsf-di-lease-rps-sweep-disabled-openloop-20260705
openLoopDurationS=20
rps=0.2,0.4,0.8,1.2
requests=4
concurrency=2
leaseEnabledMaxStableRps=0.2
noLeaseMaxStableRps=0.2
0.2Result=SUCCESS, successRate=1.0
0.4/0.8/1.2Result=FAILURE, successRate=0.5
failureReason=local-open-loop-backpressure before provider admission lease becomes the bottleneck.
interpretation=current open-loop child-process user driver must be fixed or replaced before using this sweep to claim lease benefit under high concurrency.
```

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies.
- **Foundational (Phase 2)**: Depends on Setup and blocks all user stories.
- **US1 Runtime-aware assignment**: Depends on Foundational.
- **US2 Lease conflict control**: Depends on Foundational; can proceed in parallel with US1 after shared schemas stabilize, but integration depends on ACK/selection compatibility.
- **US3 Edge-aware placement**: Depends on Foundational and can proceed in parallel with US1.
- **US4 Replan**: Depends on US2 lease rejection reasons and US1 assignment output.
- **US5 MiniNDN evidence**: Depends on US1, US2, and US3; useful after US4 for stale-state scenarios.
- **Polish**: Depends on selected user stories.

### User Story Dependencies

- **US1 (P1)**: MVP scoring slice; no dependency on US2 lease enforcement.
- **US2 (P1)**: Required for real multi-user conflict control.
- **US3 (P1)**: Required for provider-to-provider network correctness.
- **US4 (P2)**: Requires lease rejection signals from US2.
- **US5 (P2)**: Requires enough implementation to run campaigns.

### Parallel Opportunities

- T002, T003, T004 can run in parallel.
- T007 and T009 can run in parallel after T006 begins.
- US1 tests T012-T014 can be written in parallel.
- US3 tests T035-T038 can be written in parallel.
- Documentation tasks T060, T061, and T062 can be done in parallel after implementation behavior stabilizes.

## Implementation Strategy

### MVP First

1. Complete Phase 1 and Phase 2.
2. Complete US1 to prove runtime-aware assignment from ACK state.
3. Complete US2 to make provider leases authoritative under multi-user contention.
4. Complete US3 to include provider-to-provider network edge cost.
5. Stop and validate with deterministic tests before MiniNDN.

### Incremental Delivery

1. Fragment identity and runtime state.
2. Runtime-aware scoring.
3. Lease/admission validation.
4. Directed provider network matrix and edge-aware scoring.
5. Bounded replan.
6. MiniNDN campaign evidence.

### Do Not Implement In This Feature

- Dedicated planner service.
- Provider-as-planner/coordinator.
- Global lock service.
- Semantic cache matching.
- Changes that bypass NDNSF security/token/NAC-ABE checks.
