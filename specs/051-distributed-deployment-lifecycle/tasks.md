# Tasks: Distributed Deployment Lifecycle

> **Superseded; do not execute.** Specs 085 and 087 replaced this draft with a
> DI-owned lifecycle and provider-authoritative fail-closed leases. Unchecked
> boxes below are retained as historical design evidence, not pending work.

## Phase 1: Provisioning-Aware Negative ACK (C++)

- [ ] T001 Add `setProvisioningContext(deploymentId, provisioningRole, expectedReadyMs)`
  to `NativeProviderReadinessState` in `NativeProviderReadiness.hpp/cpp`.
  Store `m_deploymentId`, `m_provisioningRole`, `m_expectedReadyMs`,
  `m_provisioningStartedMs`.

- [ ] T002 Extend `makeAckDecision` payload: when `!ready && !deploymentId.empty()`,
  append `deploymentId=X;provisioningRole=Y;expectedReadyMs=Z;` to the ACK payload.

- [ ] T003 Call `setProvisioningContext` in `DI_NativeProviderExecutable.cpp`
  after `markInstalling`, passing provider name as deploymentId placeholder,
  `joinRoles(allowedRoles)` as provisioningRole, and 30000ms estimate.

- [ ] T004 Parse new ACK fields (`deploymentId`, `provisioningRole`,
  `expectedReadyMs`) in `user_driver.py`'s `ack_candidates_snapshot()`.

## Phase 2: Deployment Data Model (Python Core)

- [ ] T005 Add `DeploymentStatus` enum and `Deployment` dataclass to
  `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`:
  `deployment_id`, `plan_id`, `service_name`, `status`, `fragment_map`,
  `created_at_ms`, `updated_at_ms`.

- [ ] T006 Add `PlacementConstraint` dataclass to NDNSF core
  `pythonWrapper/ndnsf/runtime_telemetry.py`: `role_id`, `min_gpu_memory_mb`,
  `min_cpu_memory_mb`, `required_backend`, `anti_affinity`, `affinity`,
  `min_replicas`, `max_replicas`.

- [ ] T007 Add NDNSD deployment encoding/decoding helpers:
  `encode_deployments_json(deployments) → str` and
  `decode_deployments_json(json_str) → list[Deployment]`.

## Phase 3: deploy_service API (Python ServiceUser)

- [ ] T008 Add `ServiceUser.deploy_service(service, plan, *, constraints, timeout_ms)`
  in `pythonWrapper/ndnsf/service.py`:
  1. Send deploy Interest (reuse existing request_collaboration infrastructure)
  2. Collect ACKs with provisioning proposals
  3. User runs placement algorithm (Filter → Score → Pick)
  4. Reserve leases on selected providers
  5. Publish deployment via NDNSD (`provider.publish_service_info` with
     `deployments` JSON in meta)
  6. Return `Deployment`

- [ ] T009 Add `ServiceUser.discover_deployments(service) → list[Deployment]`:
  Read `get_ndnsd_services()` → parse `serviceMetaInfo["deployments"]` JSON →
  return typed Deployment objects.

- [ ] T010 Add `ServiceUser.get_deployment(deployment_id) → Deployment | None`.

## Phase 4: request_collaboration with deployment_id

- [ ] T011 Add optional `deployment_id` parameter to `ServiceUser.request_collaboration`.
  When set: pre-populate `role_provider_preference` from deployment's
  `fragment_map` before sending the request.

- [ ] T012 In the user driver, add `--deployment-id` flag and wire it through
  the child process command builders.

## Phase 5: NDNSD Deployment Broadcasting

- [ ] T013 After User locks in a deployment (leases reserved), call
  `user.update_ndnsd_meta` (or equivalent) to publish deployment state
  via the existing NDNSD heartbeat mechanism.

- [ ] T014 In the advisory coordinator, read NDNSD deployment state and
  include it in the health tracker's `provider_capacity` output as
  `activeDeployments`.

- [ ] T015 Implement deterministic convergence: when multiple users see the
  same NDNSD deployment data, they MUST compute the same deployment view.
  Use `sorted(deployments, key=lambda d: d.deployment_id)` as tie-breaking.

## Phase 6: DISK_RESIDENT Cache State

- [ ] T016 Update `FragmentResidency` enum documentation: `DISK_RESIDENT` is a
  valid warm-standby state, not "expired". `readyCost=35ms`.

- [ ] T017 In `discover_deployments()`, include DISK_RESIDENT deployments with
  a `readyCost` field, so users can decide whether to use them or deploy fresh.

- [ ] T018 When executing against a DISK_RESIDENT deployment, providers
  automatically reload from disk (existing behavior). The deployment status
  transitions from DISK_RESIDENT → ACTIVE upon successful reload.

## Phase 7: Placement Algorithm

- [ ] T019 Add `filter_feasible_providers(role, constraint, candidates) → list`
  in `runtime_v1.py`: filter by GPU memory, backend support, worker availability,
  anti-affinity, circuit breaker state.

- [ ] T020 Add `score_provider_candidates(role, feasible, fragment_state, ack_state) → list`
  in `runtime_v1.py`: score each candidate by residency cost + queue wait +
  affinity bonus + edge cost.

- [ ] T021 Add `pick_optimal_assignment(plan, scored_by_role, constraints) → fragment_map`
  in `runtime_v1.py`: for each role, pick the best-scored provider. For small
  combinatorics (≤3 roles, ≤3 candidates each), use Cartesian product with
  edge costs. For larger, use greedy with edge-cost-aware ordering.

## Phase 8: Tests

- [ ] T022 Add unit tests for `Deployment` lifecycle state transitions.
- [ ] T023 Add unit tests for `PlacementConstraint` filtering and scoring.
- [ ] T024 Add unit tests for NDNSD deployment encoding/decoding round-trip.
- [ ] T025 Add regression test: provider negative-ACK includes provisioning context.
- [ ] T026 Run all existing tests (no regressions).

## Phase 9: MiniNDN Validation

- [ ] T027 Run sequential smoke: deploy → discover → execute (2 deployments, 2 users).
- [ ] T028 Run DISK_RESIDENT smoke: deploy → evict from memory → execute and verify reload.
- [ ] T029 Run multi-user smoke: User A deploys while User B receives provisioning
  negative-ACK, waits, then User B executes against User A's deployment.
- [ ] T030 Run comparison sweep: deploy-first vs deploy-on-first-request,
  measure p50/p95 latency improvement from warm deployment.
