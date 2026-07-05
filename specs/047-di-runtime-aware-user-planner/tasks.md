# Tasks: DI Runtime-Aware User-Side Planner

**Input**: Design documents from `specs/047-di-runtime-aware-user-planner/`

**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

**Tests**: Include tests because this feature changes planner, ACK/selection semantics, lease validation, and MiniNDN evaluation behavior.

**Organization**: Tasks are grouped by independently testable user stories.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Prepare fixtures and shared schema locations without changing runtime behavior.

- [ ] T001 Create runtime-aware planner fixture directory in `examples/python/NDNSF-DistributedInference/native_di_tracer/runtime_aware_fixtures/`
- [ ] T002 [P] Add sample model fragment inventory fixture in `examples/python/NDNSF-DistributedInference/native_di_tracer/runtime_aware_fixtures/provider_fragments.json`
- [ ] T003 [P] Add sample directed provider network matrix fixture in `examples/python/NDNSF-DistributedInference/native_di_tracer/runtime_aware_fixtures/provider_network_matrix.json`
- [ ] T004 [P] Add sample multi-user workload fixture in `examples/python/NDNSF-DistributedInference/native_di_tracer/runtime_aware_fixtures/multi_user_requests.json`
- [ ] T005 Add feature flag/profile knob for runtime-aware user-side planning in `examples/di-native-tracer.runtime.json`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Define shared entities and compatibility boundaries required by all stories.

- [ ] T006 Define `ModelFragmentKey`, `FragmentRuntimeState`, `ProviderPairMetric`, and `ProviderRuntimeState` schemas in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- [ ] T007 [P] Add serialization/deserialization unit tests for runtime-aware schemas in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [ ] T008 Add `PlanTemplate` versus `RuntimeAssignment` schema separation in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- [ ] T009 [P] Add fixture loader helpers for fragment inventory and network matrix in `examples/python/NDNSF-DistributedInference/native_di_tracer/runtime_aware_fixtures/loader.py`
- [ ] T010 Document runtime-aware planner data assumptions in `docs/NDNSF-DI-runtime-workflow.md`
- [ ] T011 Run `python3 -m py_compile` for changed Python modules and `python3 tests/python/test_ndnsf_di_runtime_aware_planner.py`

**Checkpoint**: Schemas and fixtures exist; no service behavior changes yet.

---

## Phase 3: User Story 1 - Runtime-aware assignment from provider ACKs (Priority: P1) MVP

**Goal**: User-side planner selects runtime assignments from provider ACK state and fragment residency.

**Independent Test**: Deterministic fixture proves GPU-loaded, CPU-resident, disk-resident, repo-available, and missing fragments are scored in the expected order.

### Tests for User Story 1

- [ ] T012 [P] [US1] Add planner fixture test for fragment residency ordering in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [ ] T013 [P] [US1] Add planner fixture test excluding providers without valid runtime state when runtime-aware mode is required in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [ ] T014 [P] [US1] Add planner fixture test for conservative fallback when runtime-aware mode is optional in `tests/python/test_ndnsf_di_runtime_aware_planner.py`

### Implementation for User Story 1

- [ ] T015 [US1] Implement residency cost constants and score breakdown structure in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- [ ] T016 [US1] Implement runtime-aware candidate scoring helper in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- [ ] T017 [US1] Wire runtime-aware scoring into the NativeTracer user-side planner path in `examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py`
- [ ] T018 [US1] Emit selected assignment and rejected candidate reasons in NativeTracer planner output in `examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py`
- [ ] T019 [US1] Validate US1 with `python3 tests/python/test_ndnsf_di_runtime_aware_planner.py`

**Checkpoint**: User-side planner can choose assignments from runtime state without lease enforcement.

---

## Phase 4: User Story 2 - Multi-user conflict control through provider leases (Priority: P1)

**Goal**: Providers issue and validate short-lived leases so concurrent users cannot all consume the same stale resource state.

**Independent Test**: Two synthetic users race for one single-slot provider role; only valid lease consumption triggers execution.

### Tests for User Story 2

- [ ] T020 [P] [US2] Add lease manager unit test for grant, expire, consume, and release behavior in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [ ] T021 [P] [US2] Add selection validation test for expired/mismatched/already-consumed leases in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [ ] T022 [P] [US2] Add C++ or shell smoke test ensuring provider rejects invalid lease before role execution in `tests/python/test_ndnsf_di_runtime_aware_planner.py`

### Implementation for User Story 2

- [ ] T023 [US2] Implement provider lease table model in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- [ ] T024 [US2] Extend ACK runtime payload builder with `LeaseOffer` fields in `ndn-service-framework/ServiceProvider.cpp`
- [ ] T025 [US2] Extend selection payload parsing to carry lease id, role id, and fragment key in `ndn-service-framework/ServiceUser.cpp`
- [ ] T026 [US2] Implement provider-side lease validation before DI role execution in `ndn-service-framework/ServiceProvider.cpp`
- [ ] T027 [US2] Add structured lease rejection reasons to provider/user diagnostics in `ndn-service-framework/ServiceProvider.cpp` and `ndn-service-framework/ServiceUser.cpp`
- [ ] T028 [US2] Validate US2 with lease unit tests and focused provider selection smoke test

**Checkpoint**: Concurrent users can receive independent ACKs, but provider lease validation controls execution.

---

## Phase 5: User Story 3 - Edge-aware placement using provider-to-provider network metrics (Priority: P1)

**Goal**: Planner scores dependency edges using directed provider-to-provider bandwidth, RTT, loss, jitter, staleness, and confidence.

**Independent Test**: Fixture proves edge-aware scoring avoids a poor provider-pair dependency edge even when compute-only ranking prefers it.

### Tests for User Story 3

- [ ] T029 [P] [US3] Add directed provider-pair metric parsing test in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [ ] T030 [P] [US3] Add edge-cost scoring test for bandwidth/RTT/loss/jitter in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [ ] T031 [P] [US3] Add stale/unknown metric penalty test in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [ ] T032 [P] [US3] Add graph assignment test where edge-aware placement beats compute-only placement in `tests/python/test_ndnsf_di_runtime_aware_planner.py`

### Implementation for User Story 3

- [ ] T033 [US3] Implement `ProviderNetworkMatrix` helper in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- [ ] T034 [US3] Implement dependency edge byte-size and transfer-cost estimator in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- [ ] T035 [US3] Integrate node-cost plus edge-cost assignment scoring in `examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py`
- [ ] T036 [US3] Add edge-cost details to selected assignment output in `examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py`
- [ ] T037 [US3] Validate US3 with graph assignment fixture tests

**Checkpoint**: Planner treats NDNSF-DI as graph placement, not independent provider ranking.

---

## Phase 6: User Story 4 - Replan after stale runtime state (Priority: P2)

**Goal**: User-side planner performs bounded replan when selected leases/providers become invalid before or during execution.

**Independent Test**: Forced lease rejection triggers replan and produces a structured `ReplanRecord`.

### Tests for User Story 4

- [ ] T038 [P] [US4] Add replan record serialization test in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [ ] T039 [P] [US4] Add bounded replan test for `FRAGMENT_EVICTED` in `tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [ ] T040 [P] [US4] Add max-attempt failure test with structured planner reason in `tests/python/test_ndnsf_di_runtime_aware_planner.py`

### Implementation for User Story 4

- [ ] T041 [US4] Implement `ReplanRecord` model and exclusion list handling in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- [ ] T042 [US4] Add bounded replan loop to NativeTracer user-side planner path in `examples/python/NDNSF-DistributedInference/native_di_tracer/user_driver.py`
- [ ] T043 [US4] Emit replan count and reason into planner metrics in `examples/python/NDNSF-DistributedInference/native_di_tracer/user_driver.py`
- [ ] T044 [US4] Validate US4 with fixture tests

**Checkpoint**: Runtime-aware user-side planning recovers from stale leases within bounded attempts.

---

## Phase 7: User Story 5 - Evidence through MiniNDN multi-user campaigns (Priority: P2)

**Goal**: MiniNDN campaigns compare static user-side planning against runtime-aware lease and edge-aware planning.

**Independent Test**: One documented MiniNDN command emits latency, lease, residency, edge-cost, replan, and utilization evidence.

### Tests for User Story 5

- [ ] T045 [P] [US5] Add dry-run test for multi-user runtime-aware campaign arguments in `tests/python/test_ndnsf_di_runtime_aware_campaign.py`
- [ ] T046 [P] [US5] Add parser test for planner metrics aggregation in `tests/python/test_ndnsf_di_runtime_aware_campaign.py`

### Implementation for User Story 5

- [ ] T047 [US5] Add multi-user workload mode to `Experiments/NDNSF_DI_NativeTracer_Minindn.py`
- [ ] T048 [US5] Add asymmetric provider-to-provider link profile to `Experiments/Topology/AI_Lab.conf` or a new DI topology fixture
- [ ] T049 [US5] Add runtime-aware planner mode flags to `tools/ndnsf_runtime.py di run` passthrough documentation and profile defaults
- [ ] T050 [US5] Emit planner metrics JSON/CSV from MiniNDN campaign in `Experiments/NDNSF_DI_NativeTracer_Minindn.py`
- [ ] T051 [US5] Add campaign summary fields for p50/p95 latency, success rate, utilization, lease counters, residency counters, edge-cost summary, and replan count in `Experiments/NDNSF_DI_NativeTracer_Minindn.py`
- [ ] T052 [US5] Run a short MiniNDN validation campaign and save canonical command/results summary under `docs/` or tracked experiment documentation

**Checkpoint**: Feature has measurable MiniNDN evidence for multi-user and provider-pair network behavior.

---

## Final Phase: Polish & Cross-Cutting Concerns

**Purpose**: Documentation, security checks, and regression cleanup.

- [ ] T053 [P] Update `docs/NDNSF-DI-runtime-workflow.md` with runtime-aware planner workflow and metrics outputs
- [ ] T054 [P] Update `docs/native-di-roadmap.md` with plan-template versus runtime-assignment and edge-aware planning rationale
- [ ] T055 Run focused token/security regressions covering UserToken/ProviderToken and selection validation paths
- [ ] T056 Run `python3 tests/python/test_ndnsf_di_runtime_aware_planner.py`
- [ ] T057 Run `python3 tests/python/test_ndnsf_di_runtime_aware_campaign.py`
- [ ] T058 Run `python3 tools/ndnsf_runtime.py di validate`
- [ ] T059 Run `git diff --check`
- [ ] T060 Record final validation commands and known limitations in `specs/047-di-runtime-aware-user-planner/quickstart.md`

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
- US3 tests T029-T032 can be written in parallel.
- Documentation tasks T053 and T054 can be done in parallel after implementation behavior stabilizes.

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
