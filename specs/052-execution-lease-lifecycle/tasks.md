# Tasks: Execution Lease Lifecycle

> **Superseded; do not execute.** Spec 085 implements the accepted
> provider-local lease authority. Unchecked boxes below document the rejected
> Merge-Provider/ref-count design and are not pending work.

## Phase 1: Core Types

- [ ] T001 Add `ExecutionLease` dataclass to `runtime_telemetry.py`: lease_id, deployment_id, user, acquired_at_ms, expires_at_ms, released.
- [ ] T002 Add `DeploymentLeaseTable` class to `runtime_v1.py`: grant, release, active_count, expire_stale.
- [ ] T003 Add `ref_count` field to `Deployment` dataclass, computed from `DeploymentLeaseTable.active_count()`.

## Phase 2: API

- [ ] T004 Add `ServiceUser.acquire_execution_lease(deployment_id, ttl_ms=30000) → dict` to `service.py`.
- [ ] T005 Add `ServiceUser.release_execution_lease(lease_id) → dict` to `service.py`.
- [ ] T006 Pass `execution_lease_id` through `request_collaboration` to provider ACK payload.

## Phase 3: Eviction Guard

- [ ] T007 Update `ServiceUser.evict_deployment(deployment_id)` to check ref_count: if >0, reject with `DEPLOYMENT_IN_USE`.
- [ ] T008 Include `ref_count` in NDNSD deployment broadcast so all users see it.

## Phase 4: Merge Provider GC

- [ ] T009 Add `DeploymentGc` class to `runtime_v1.py`: periodic scan, expire stale leases, update ref_count, transition IDLE deployments to DISK_RESIDENT.
- [ ] T010 Integrate `DeploymentGc` into advisory coordinator or as standalone service.

## Phase 5: Tests + Validation

- [ ] T011 Unit tests: lease acquire/release round-trip, ref_count computation, eviction guard.
- [ ] T012 Unit tests: GC expiration, IDLE → DISK_RESIDENT transition.
- [ ] T013 Run all existing tests (no regressions).
- [ ] T014 MiniNDN smoke: deploy → acquire lease → execute → release → verify ref_count.
- [ ] T015 MiniNDN smoke: attempt evict while lease active → verify DEPLOYMENT_IN_USE rejection.
