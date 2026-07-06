# Implementation Plan: Core Coordination Envelope

**Branch**: `049-core-coordination-envelope` | **Date**: 2026-07-05 |
**Spec**: [spec.md](spec.md)

## Summary

Move service-neutral coordination concepts into the NDNSF Python core package
and make NDNSF-DI advisory coordination use those concepts as an application
payload. The core owns intent/suggestion freshness, nonce, stable digest,
window id, proof, and opaque payload schema. NDNSF-DI owns plan templates,
role assignments, fragment scoring, edge costs, and merge rules.

This iteration also adds a Python service transport MVP: coordination requests
and responses are encoded as JSON payloads carried by the existing NDNSF
dynamic service API. NativeTracer can now start an advisory coordinator service
and compare pure user-side planning against advisory-coordinator planning in
the RPS sweep harness.

## Technical Context

**Language**: Python 3 for the first reusable contract.

**Core Files**:

- `pythonWrapper/ndnsf/coordination.py`
- `pythonWrapper/ndnsf/__init__.py`
- `tests/python/test_ndnsf_core_coordination.py`

**DI Files**:

- `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- `NDNSF-DistributedInference/ndnsf_distributed_inference/__init__.py`
- `tests/python/test_ndnsf_di_advisory_coordinator.py`
- `examples/python/NDNSF-DistributedInference/native_di_tracer/advisory_coordinator.py`
- `examples/python/NDNSF-DistributedInference/native_di_tracer/user_driver.py`
- `Experiments/NDNSF_DI_NativeTracer_Minindn.py`
- `Experiments/NDNSF_DI_RuntimeAware_RpsSweep.py`

## Boundary Decision

NDNSF core owns:

- `CoordinationIntent`
- `CoordinationSuggestion`
- `CoordinationWindow`
- `CoordinationMode`
- stable digest / freshness / proof helpers
- opaque `payload_schema` and `payload`
- JSON request/response encoding for the Python dynamic service path
- service wrapper names and payload decode/encode errors

NDNSF-DI owns:

- `PlanIntent` as a DI wrapper over `CoordinationIntent`
- `AdvisorySuggestion` as a DI wrapper over `CoordinationSuggestion`
- DI role assignments and provider scoring
- validation that suggested providers are valid under current DI ACK metadata
  and leases
- MiniNDN workload interpretation and measured comparison of pure vs advisory
  planning

## Multi-User Scheduling Boundary

Multiple users should coordinate when they compete for the same provider pool,
but users should not directly negotiate with each other. NDNSF core provides
the reusable mechanisms: opaque coordination intents and suggestions, generic
runtime hints, admission leases, structured rejection reasons, freshness/proof
helpers, and selection-time lease validation. These mechanisms are useful for
DI, UAV, repository, and future workflow applications.

NDNSF-DI owns the scheduling policy on top of those mechanisms. The DI payload
may include role candidates, model fragment identity, fragment residency,
optional lease offers, runtime hints, estimated role duration, ready cost, and
provider-to-provider transfer costs. The advisory coordinator may use these
fields to reserve providers across a rolling multi-user window and to return a
suggested role assignment, but the suggestion is advisory only. The user must
still filter the suggestion against current ACKs and valid leases, and the
provider remains the final authority by validating the selected lease before
execution.

This keeps the core service framework reusable while allowing NDNSF-DI to
optimize the distributed inference objective: high provider utilization, fewer
lease conflicts, lower p50/p95 latency, and better fragment/cache reuse.

## Runtime Hint Snapshot Integration

The first wire-path integration uses a DI-owned runtime hint snapshot. The
NativeTracer MiniNDN harness writes `runtime-hints.json` from the provider role
profiles, current admission settings, estimated role duration, ready cost, and
fragment residency. The user driver reads that file and enriches
`roleCandidates` inside the DI payload before sending the generic
`CoordinationIntent`.

The NDNSF core envelope remains unchanged. It still carries an opaque payload;
the names `runtimeHint`, `leaseOffers`, `estimatedDurationMs`, `readyCostMs`,
and `residency` are NativeTracer/NDNSF-DI scheduling fields. The advisory
coordinator may use them to score candidates, but the provider still validates
admission leases at execution time, and users still treat the suggestion as
advisory.

The current snapshot is generated before the MiniNDN run starts, so it proves
the data path and scoring contract. The next step is to replace the harness
snapshot source with live provider telemetry published by the provider runtime:
queue depth, active workers, loaded fragments, GPU/CPU memory pressure, and
provider-pair network metrics.

The next incremental step refreshes that snapshot after provider provisioning.
Native providers already emit `NDNSF_DI_FRAGMENT_INVENTORY` events when model
fragments become resident or are observed during execution. The MiniNDN harness
now parses those real provider logs after `NDNSF_DI_NATIVE_PROVIDER_PROVISION_READY`
and rewrites `runtime-hints.json` before starting the user driver. This keeps
the same DI payload contract while moving residency, fragment digest, backend,
artifact path, and sample age from static profile assumptions to observed
provider runtime state.

Native provider ACKs already carry runtime capacity fields such as queue depth,
ready queue, waiting inputs, active workers, worker count, idle workers,
runtime status, admission result, and admission lease identifiers. The next
minimal bridge records those ACK payloads in structured provider logs and
aggregates them into `providerAckRuntimeHints` in the MiniNDN summary. This
does not yet make the advisory coordinator consume live ACK candidates before
planning, but it proves that provider-side runtime telemetry is emitted on the
real service invocation path and can be collected without changing NDNSF core.

The collaboration ACK observer closes the next API gap. Python
`ServiceUser.request_collaboration` now accepts an optional `ack_observer`
callback. The callback receives the same `AckCandidate` shape used by
`request_service_select`, before the native role-assignment selector chooses
providers. This keeps selection authority in the existing native path while
allowing NDNSF-DI user/advisory logic to record live ACK queue, worker,
runtime, lease, and network telemetry snapshots for future planning windows.

## Validation

```bash
PYTHONPATH=pythonWrapper python3 tests/python/test_ndnsf_core_coordination.py
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference python3 tests/python/test_ndnsf_di_advisory_coordinator.py
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference python3 tests/python/test_ndnsf_di_runtime_aware_planner.py
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference python3 tests/python/test_ndnsf_di_runtime_v1.py
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:Experiments python3 tests/python/test_ndnsf_di_runtime_aware_campaign.py
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:Experiments python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --dry-run --advisory-coordinator --requests 1 --concurrency 1
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:Experiments python3 Experiments/NDNSF_DI_RuntimeAware_RpsSweep.py --dry-run --compare-advisory-coordinator --out /tmp/ndnsf-di-rps-sweep-advisory-dry-run --rps 0.2 --requests 2 --concurrency 2
```

## Constitution Check

- **Canonical Dynamic Runtime**: PASS. No generated stub path is introduced.
- **Security Is Part Of The Data Path**: PASS. Proof/freshness is shared, and
  provider authority remains with leases/tokens/permissions.
- **CodeGraph First, Source Verified**: PASS. CodeGraph was used before edits.
- **Spec-Driven Changes For Durable Work**: PASS. SPEC049 records the boundary.
- **Verify With The Right Scope**: PASS. Unit tests cover the Python contract
  and dry-run checks cover MiniNDN command wiring. Full MiniNDN measured
  comparison remains the next evidence step.
