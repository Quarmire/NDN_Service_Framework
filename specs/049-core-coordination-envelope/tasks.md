# Tasks: Core Coordination Envelope

## Phase 1: Boundary Design

- [x] T001 Define which coordination fields belong in NDNSF core in `specs/049-core-coordination-envelope/spec.md`.
- [x] T002 Define which DI planning fields remain in NDNSF-DI in `specs/049-core-coordination-envelope/plan.md`.
- [x] T003 Use DeepSeek for a second-pass boundary checklist and keep Codex responsible for final code review.

## Phase 2: Core Contract

- [x] T004 Add `CoordinationMode`, `CoordinationIntent`, `CoordinationWindow`, and `CoordinationSuggestion` in `pythonWrapper/ndnsf/coordination.py`.
- [x] T005 Add shared stable digest, freshness, proof, and verification helpers in `pythonWrapper/ndnsf/coordination.py`.
- [x] T006 Export the core coordination API from `pythonWrapper/ndnsf/__init__.py`.

## Phase 3: DI Integration

- [x] T007 Make `PlanIntent` extend the generic `CoordinationIntent` in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`.
- [x] T008 Make `AdvisorySuggestion` extend the generic `CoordinationSuggestion` in `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`.
- [x] T009 Keep DI role assignment payload and scoring in NDNSF-DI.
- [x] T010 Preserve existing DI advisory API names and test expectations.

## Phase 4: Tests and Validation

- [x] T011 Add core coordination tests in `tests/python/test_ndnsf_core_coordination.py`.
- [x] T012 Run DI advisory coordinator tests.
- [x] T013 Run existing runtime-aware planner and runtime-v1 tests.
- [x] T014 Run `git diff --check` and Python compile checks.
- [x] T015 Record future work: C++ wire protocol, signed controller proof, and MiniNDN coordinator service validation.

## Phase 5: NDNSF Service Wire Path

- [x] T016 Add `CoordinationRequest` and `CoordinationResponse` payload
  wrappers in `pythonWrapper/ndnsf/coordination.py`.
- [x] T017 Add JSON encode/decode helpers for coordination request/response
  payloads.
- [x] T018 Add `CoordinationServiceProvider` and `CoordinationServiceClient`
  wrappers over the existing dynamic service API.
- [x] T019 Add a NativeTracer advisory coordinator executable that registers
  `/NDNSF/Coordination/Advisory`.
- [x] T020 Make the NativeTracer user driver request advisory suggestions
  through the NDNSF service path when `--coordination-service` is set.
- [x] T021 Add MiniNDN harness support for `--advisory-coordinator`, including
  coordinator identity, policy, routing, and process startup.
- [x] T022 Add RPS sweep support for pure user-side vs advisory-coordinator
  comparison modes.
- [x] T023 Add tests for request/response payload round trip, provider/client
  service wrapper shape, harness dry-run wiring, and RPS comparison command
  generation.
- [x] T024 Run focused Python tests and dry-run harness checks.
- [x] T025 Run measured MiniNDN pure/advisory comparison and record conflict
  rate, p50/p95, and stable RPS.

## Phase 6: Contention Evidence and Assignment Payload

- [x] T026 Run a high-contention MiniNDN comparison with open-loop load,
  provider admission limits, and role execution delay.
- [x] T027 Record that wire-path-only advisory suggestions do not improve
  contention when `roleAssignments` is empty.
- [x] T028 Pass the harness assignment CSV to the user driver and include
  role/provider assignments in coordination intents.
- [x] T029 Verify in MiniNDN logs that `/NDNSF/Coordination/Advisory` returns
  non-empty role assignments over the real service path.
- [x] T030 Make the coordinator compute or rewrite role assignments from
  multi-user windows instead of echoing the user's current assignment; include
  a deterministic window id/version so users can reject stale suggestions.
- [x] T031 Make the user merge valid advisory role assignments into provider
  selection before collaboration execution, after rechecking that suggested
  providers are still valid candidates.
- [x] T032 Add regression tests for non-empty role assignments, malformed or
  stale suggestions, and multi-user windows that must avoid overlapping
  provider assignments when alternatives exist.

## Phase 7: Capacity-Pool Candidate Evidence

- [x] T033 Add a capacity-pool assignment fixture with one primary provider and
  one alternate provider for every NativeTracer role.
- [x] T034 Pass the assignment CSV into open-loop and process-pool user workers
  so child requests carry `roleCandidates` in advisory coordination intents.
- [x] T035 Add regression tests for capacity-pool candidate generation and RPS
  sweep command generation.
- [x] T036 Run a MiniNDN advisory capacity-pool smoke and verify that different
  concurrent requests receive different provider assignments through the real
  NDNSF coordination service path.
- [x] T037 Run a pure user-side versus advisory-coordinator capacity-pool sweep
  and record the measured result, including the current negative finding that
  advisory coordination does not yet raise max stable RPS under this overload.

## Phase 8: Overload Fast-Fail Boundary

- [x] T038 Add a user-driver overload fast-fail timeout knob that uses a shorter
  collaboration timeout for overload experiments while preserving the base
  timeout for normal runs.
- [x] T039 Record per-request `overloadFastFail` and workload-level
  `overloadFastFailCount` diagnostics.
- [x] T040 Pass the overload fast-fail knob through the MiniNDN harness and RPS
  sweep wrapper, and make the parent wait deadline follow the effective
  timeout.
- [x] T041 Add regression tests for user-driver fast-fail metadata and dry-run
  command generation.
- [x] T042 Run a MiniNDN overload comparison with and without fast-fail enabled
  and record whether it reduces failed-request latency without changing the
  underlying stable RPS boundary.

## Phase 9: Multi-User Lease-Aware Advisory Scheduling

- [x] T043 Define the NDNSF/NDNSF-DI boundary for multi-user resource
  coordination in `specs/049-core-coordination-envelope/plan.md`.
- [x] T044 Upgrade the wire-path NativeTracer advisory coordinator in
  `examples/python/NDNSF-DistributedInference/native_di_tracer/advisory_coordinator.py`
  from least-used balancing to DI-owned rolling reservation and optional
  lease/runtime-hint scoring.
- [x] T045 Keep all model, fragment, role, lease-offer scoring, and
  provider-reservation semantics inside the DI payload layer rather than
  adding them to NDNSF core.
- [x] T046 Add regression tests proving the coordinator avoids reserved busy
  providers when another candidate exists and prefers earlier lease/runtime
  availability.
- [x] T047 Run focused tests and static checks for the advisory coordinator,
  user driver, harness, and sweep wrappers.

## Phase 10: Runtime Hint Snapshot Integration

- [x] T048 Add a NativeTracer user-driver input for provider runtime and lease
  hint snapshots without changing the NDNSF core coordination envelope.
- [x] T049 Enrich DI `roleCandidates` with optional `runtimeHint`,
  `leaseOffers`, `estimatedDurationMs`, `readyCostMs`, and fragment residency
  before sending the coordination intent.
- [x] T050 Generate a MiniNDN harness `runtime-hints.json` snapshot from the
  current provider role profiles and admission settings.
- [x] T051 Pass the runtime hint snapshot through dry-run, child-worker, and
  real MiniNDN advisory-coordinator commands.
- [x] T052 Add regression tests for runtime hint enrichment, snapshot
  generation, and dry-run command wiring.

## Phase 11: Provider Runtime Inventory Refresh

- [x] T053 Refresh `runtime-hints.json` after provider provisioning by parsing
  real provider `NDNSF_DI_FRAGMENT_INVENTORY` logs.
- [x] T054 Update candidate fragment residency, digest, backend, artifact path,
  sample age, and source from observed provider runtime inventory.
- [x] T055 Start the user driver only after provider inventory is observed and
  the refreshed runtime hint snapshot is written in advisory-coordinator
  MiniNDN runs.
- [x] T056 Add regression coverage for provider-inventory refresh of runtime
  hints.

## Phase 12: Provider ACK Runtime Telemetry Evidence

- [x] T057 Include provider identity and ACK payload fields in
  `NDNSF_DI_NATIVE_PROVIDER_ACK_DECISION` logs.
- [x] T058 Aggregate provider ACK runtime telemetry from MiniNDN logs into
  `providerAckRuntimeHints` in the run summary.
- [x] T059 Extract queue, ready queue, waiting inputs, active workers, workers,
  idle workers, runtime status, and lease identifiers from ACK payloads.
- [x] T060 Add regression coverage for ACK runtime telemetry aggregation.

## Phase 13: Collaboration ACK Observer API

- [x] T061 Add an optional Python `ack_observer` callback to
  `ServiceUser.request_collaboration`.
- [x] T062 Reuse the existing `AckCandidate` shape so observers receive
  provider, service, request id, status, message, payload, and network
  telemetry.
- [x] T063 Invoke the observer inside the native collaboration role-selection
  policy before built-in provider selection.
- [x] T064 Record NativeTracer per-request ACK candidate snapshots from the
  observer, including queue, worker, runtime, and lease fields.
- [x] T065 Add regression coverage for ACK candidate snapshot parsing.
