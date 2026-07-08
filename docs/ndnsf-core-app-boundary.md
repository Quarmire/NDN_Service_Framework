# NDNSF Core/App Boundary

This document records which mechanisms belong in the reusable NDNSF core and
which mechanisms remain inside NDNSF-DistributedRepo, NDNSF-UAV-APP, and
NDNSF-DistributedInference.

## Boundary Principle

NDNSF core owns service-neutral network mechanisms:

- service invocation: request, ACK, selection, response, Targeted invocation,
  and trusted local invocation;
- security and bootstrap: controller-signed permissions, NAC-ABE routing,
  one-time tokens, certificate bootstrap, and negative ACK reason codes;
- reusable data movement surfaces: exact-name large-data retrieval and the
  continuous-publication stream substrate;
- reusable runtime coordination: provider capability hints, operation status,
  admission leases, coordination envelopes, stream health, and network
  telemetry.

Applications own domain semantics:

- what a service means;
- how an object, mission, model, video frame, or tensor is interpreted;
- how to score domain-specific candidates;
- how to transform or execute application payloads.

The core should provide envelopes and validation helpers. Applications should
put domain-specific fields in typed payloads inside those envelopes.

## Core Mechanisms

### Provider Capability and Runtime Hints

Core owns the app-neutral answer to:

```text
Can this provider accept this service request now, and what is its current
runtime condition?
```

Reusable Python helpers:

- `ProviderCapabilityHint`
- `RuntimeHint` / `GenericProviderRuntimeHint`
- `GenericAckMetadata`
- `GenericAdmissionLease`
- `ProviderAdmissionLeaseTable`
- `RejectionReason`

Applications may attach service-specific payloads through
`service_payload_schema` and `service_payload`. For example, DI can attach model
fragment residency; Repo can attach storage capacity; UAV can attach camera
readiness. The core does not interpret those payloads.

### Service Operation Status

Core owns the app-neutral lifecycle of long-running work:

```text
QUEUED -> RUNNING / WAITING_INPUT -> DONE / FAILED / CANCELED / EXPIRED
```

Reusable Python helper:

- `ServiceOperationStatus`

Repo insert/fetch operations, UAV missions and recordings, and DI provisioning
or execution may all expose this shape. App-specific results belong in
`result_reference` or `metadata`.

### Stream Health

Core owns the continuous-publication stream substrate:

- `StreamInfo`
- `StreamChunk`
- `StreamFecInfo`
- `StreamProducerBuffer`
- `StreamConsumerReorderBuffer`
- `StreamAdaptiveFetcherState`
- `StreamHealth`

Core stream helpers report session identity, stale sessions, gaps, duplicates,
backlog pressure, and adaptive fetch decisions. Applications still own codecs,
ROI logic, video bitrate policy, tensor semantics, and the actual FEC repair
algorithm.

Use streams for live or near-live continuous data. Use exact-name large-data
retrieval for complete named objects such as files, recordings, model
artifacts, manifests, and planned DI tensor bundles.

### Provider-Pair Telemetry

Core owns reusable network measurements between providers:

- `PeerNetworkMetric`
- `ProviderNetworkMatrix`

The matrix can estimate transfer cost and rank provider-pair candidates for
selection or cooperation. Applications decide how much those costs matter. DI
may use them for dependency exchange planning; Repo may use them for replica
placement; UAV may use them for multi-drone coordination.

## NDNSF-DistributedRepo Boundary

Repo owns:

- repo object manifests and catalog semantics;
- storage backends and persistence modes;
- replica placement policy;
- tombstones, TTL, catalog repair, and storage-specific status;
- STORE, FETCH, MANIFEST, INVENTORY, and catalog operations.

Repo should use core envelopes for:

- provider storage capability ACKs;
- operation status;
- admission or overload rejection;
- provider-pair telemetry for replica placement;
- exact-name large-data references for signed segmented Data.

Repo should not move its catalog or storage backend into core.

## NDNSF-UAV-APP Boundary

UAV owns:

- MAVLink and flight-controller semantics;
- mission planning, safety checks, and progress interpretation;
- camera readiness, H264/H265, ROI, YOLO, decoder queues, and bitrate policy;
- the actual FEC repair algorithm;
- ground-station GUI and operator workflows.

UAV should use core envelopes for:

- stream identity, chunk metadata, and stream health;
- command/mission operation status;
- readiness/capability ACKs;
- recording or data-product references through exact-name large-data paths.

UAV should not move MAVLink, camera, codec, ROI, or mission semantics into core.

## NDNSF-DistributedInference Boundary

DI owns:

- model split planning, stages, shards, roles, and merge logic;
- model fragment identity and residency semantics;
- exact forward cache, semantic cache, KV cache, and long-context policy;
- ONNX/LLM runtime backends and tensor bundle codecs;
- DI-specific advisory scheduling policy.

DI should use core envelopes for:

- provider capability and runtime hints;
- admission leases and overload fast-fail;
- operation status for provisioning and execution;
- provider-pair telemetry for dependency exchange;
- coordination intents/suggestions for advisory multi-user planning.

DI should not move model-specific planner logic, cache semantics, or tensor
formats into core.

## Missing or Incomplete Core Surfaces

The current implementation now has the core vocabulary and the first app
bridges, but some migrations are still incremental.

Completed bridge points:

- Repo ACK payloads keep the legacy storage fields and also carry
  `ProviderCapabilityHint` with storage capacity in `service_payload`.
- Repo `CAPABILITY` responses include `ProviderCapabilityHint`; main store and
  insert paths expose `ServiceOperationStatus` and, when a named object is
  produced, `DataProductReference`.
- DI provider ACKs carry `ProviderCapabilityHint` for ready, unavailable, and
  admission-rejected providers while keeping existing ACK fields.
- DI artifact provisioning ACKs carry `ServiceOperationStatus` for runtime
  install/materialization progress.
- DI `ProviderFragmentInventoryManager` can produce a core
  `ProviderCapabilityHint` from observed fragment residency.
- DI runtime-aware scoring can consume `ProviderCapabilityHint` directly as
  runtime metadata, so provider selection no longer requires DI-only
  `genericAckMetadata` when the core envelope is present.
- NativeTracer MiniNDN summaries decode typed DI provider ACK envelopes into
  `coreEnvelopeSummary`, including provider readiness, reason codes,
  service-payload schemas, operation states, and latest provider runtime view.
- DI `runtime_v1.py` now reuses the core telemetry classes for generic ACK
  metadata, runtime hints, admission leases, provider-pair metrics, and
  provider network matrices instead of maintaining duplicate local
  definitions.
- Repo Python clients prefer `ProviderCapabilityHint` over conflicting legacy
  ACK fields while retaining legacy-only fallback.
- Core C++ and Python stream helpers expose `StreamHealth`; UAV can map its
  video adaptive state to this helper while retaining H264/FEC/ROI policy.

Remaining migrations:

- Repo, UAV, and DI should gradually emit `ProviderCapabilityHint` instead of
  one-off ACK fields where practical.
- Repo operation status should be mapped to `ServiceOperationStatus`.
- UAV mission/recording status should be mapped to `ServiceOperationStatus`
  without changing MAVLink or camera logic.
- DI provisioning/execution status should keep expanding
  `ServiceOperationStatus` coverage while keeping model-specific details in DI
  payloads.
- Provider-pair telemetry should be collected from live runtime measurements
  and fed into app-level selection/cooperation policies.
- UAV still surfaces domain-specific adaptive-video fields in its GUI; the
  core `StreamHealth` mapping is now available but the display migration should
  be done separately to avoid mixing UI changes into the boundary layer.

These migrations should be done one workload at a time with regression tests.
