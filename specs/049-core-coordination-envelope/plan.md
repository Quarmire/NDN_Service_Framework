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

## Core/App Boundary Envelopes

The next boundary extension makes reusable runtime concepts explicit without
moving application semantics into the core. NDNSF core owns provider
capability hints, runtime hints, rejection reason vocabulary, service operation
status, stream health, and provider-pair telemetry ranking. Repo, UAV, and DI
own their domain payloads and policies.

The Python core exposes `ProviderCapabilityHint`, `RuntimeHint`,
`RejectionReason`, `ServiceOperationStatus`, `StreamHealth`, and
`ProviderNetworkMatrix` candidate ranking helpers. These are intentionally
service-neutral envelopes:

- Repo may put storage capacity and catalog details in `service_payload`.
- UAV may put camera, mission, or recording details in `service_payload`.
- DI may put model fragment, cache, and role details in `service_payload`.

Applications should migrate to these envelopes incrementally. The core must
not absorb Repo catalog semantics, UAV MAVLink/video policy, or DI model split
and cache policy.

The first Repo/UAV/DI migration bridge keeps existing wire fields source- and
payload-compatible while adding typed core envelopes beside them. Repo storage
ACKs now carry `ProviderCapabilityHint`, Repo store/insert responses can expose
`ServiceOperationStatus` and `DataProductReference`, DI provider ACKs carry
`ProviderCapabilityHint`, DI artifact provisioning ACKs carry
`ServiceOperationStatus`, and DI fragment inventory can produce a core
capability hint from observed residency. The stream substrate now has a C++
`StreamHealth` snapshot helper matching the Python helper, so UAV stream
consumers can report generic stream condition without moving H264/FEC/ROI
policy into NDNSF core.

This is a bridge migration rather than a hard protocol replacement. Legacy
Repo/DI payload fields remain available so existing clients and experiment
parsers keep working. New code should prefer the typed core envelopes, and a
later cleanup can remove local duplicate telemetry dataclasses after all app
consumers have regression coverage.

The second bridge migration moves selected consumers to core-first parsing.
Repo Python clients now treat `ProviderCapabilityHint` as authoritative when
it is present and use legacy storage fields only as fallback. DI
runtime-aware candidate scoring can derive `GenericAckMetadata` from
`ProviderCapabilityHint`, so a provider that emits only the core capability
envelope can still participate in DI runtime-aware planning. UAV
`VideoAdaptiveState` now maps to core `StreamHealth`, giving the ground-station
runtime a reusable stream-health snapshot without changing video bitrate,
decoder, FEC, or ROI policy.

The DI runtime cleanup removes the local duplicate definitions of generic ACK
metadata, runtime hints, admission leases, peer metrics, and provider network
matrices from `runtime_v1.py`. The public names remain importable from
`ndnsf_distributed_inference.runtime_v1`, but they now point to the reusable
core types in `ndnsf.runtime_telemetry`. Serialization remains JSON/dict based;
no NDNSF-DI pickle path depends on the old local class module names.

The experiment visibility bridge makes those core envelopes directly visible
in NativeTracer MiniNDN outputs. Provider ACK logs still keep the legacy
runtime fields for old parsers, but the harness now decodes typed
`ProviderCapabilityHint` and nested `ServiceOperationStatus` payloads into
`coreEnvelopeSummary`. The Qwen MiniNDN GUI/headless path passes the same
summary through unchanged, so the GUI and CLI experiment paths report the same
provider readiness, reason-code, schema, and latest-provider evidence.

The non-headless Qwen MiniNDN tab now renders that evidence directly. It keeps
the experiment command path unchanged, reads the same `summary.json`, and shows
provider readiness, reason codes, operation states, latest provider runtime
view, and the legacy ACK runtime hint counters. Operators can refresh the panel
from the current output directory without rerunning MiniNDN.

The C++ NativeTracer provider now emits the same typed provider capability
envelope on the real ACK path. The ACK payload keeps the old
`roles=...;queue=...;runtimeStatus=...;` fields for existing parsers and adds
`providerCapabilityHint=json64:<json>` with provider name, service name,
readiness, runtime queue/active-work counts, and DI capability payload schema.
This closes the gap between Python provider tests and real Qwen MiniNDN runs:
`coreEnvelopeSummary` is now populated by actual native provider ACKs.

## Validation

```bash
PYTHONPATH=pythonWrapper python3 tests/python/test_ndnsf_core_coordination.py
PYTHONPATH=pythonWrapper python3 tests/python/test_ndnsf_core_boundary_envelopes.py
PYTHONPATH=.:pythonWrapper:NDNSF-DistributedInference:NDNSF-DistributedRepo/pythonWrapper python3 tests/python/test_ndnsf_app_core_envelope_migration.py
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference python3 tests/python/test_ndnsf_di_advisory_coordinator.py
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference python3 tests/python/test_ndnsf_di_runtime_aware_planner.py
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference python3 tests/python/test_ndnsf_di_runtime_v1.py
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference:Experiments python3 tests/python/test_ndnsf_di_runtime_aware_campaign.py
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference python3 tests/python/test_ndnsf_di_tk_gui.py
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
