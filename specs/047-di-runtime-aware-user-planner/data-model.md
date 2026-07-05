# Data Model: DI Runtime-Aware User-Side Planner

## PlanIntent

Represents the user request before assignment.

Fields:
- `requestId`
- `requester`
- `serviceName`
- `modelId`
- `modelVersion`
- `modelDigest`
- `inputRefs`
- `contextTokens`
- `generatedTokens`
- `latencyTargetMs`
- `maxReplanAttempts`
- `requiredBackends`

Validation:
- `requestId`, `serviceName`, and `modelId` are required.
- `contextTokens` and `generatedTokens` must be non-negative.
- `maxReplanAttempts` must be bounded.

## PlanTemplate

Reusable split plan independent of runtime provider assignment.

Fields:
- `templateId`
- `modelId`
- `modelDigest`
- `splitStrategy`
- `roles`
- `dependencies`
- `estimatedRoleCosts`
- `validProviderConstraints`

Validation:
- Role ids are unique.
- Dependencies reference existing roles.
- Fragment keys in roles must match model and split metadata.

## RuntimeAssignment

Per-request mapping from roles to provider leases.

Fields:
- `requestId`
- `templateId`
- `roleAssignments`
- `selectedAt`
- `scoreBreakdown`
- `replanAttempt`

Validation:
- Every required role has exactly one selected provider/lease.
- Selected leases must be unexpired at selection time.
- Dependency edges must have an estimated edge cost.

## GenericAckMetadata

NDNSF core envelope for structured provider metadata carried by ACKs.

Fields:
- `schema`
- `providerRuntimeHint`
- `leaseOffers`
- `servicePayloadSchema`
- `servicePayload`
- `metricDigest`
- `notes`

Validation:
- Provider identity must match the ACK provider.
- Core validates generic fields only.
- Service-specific payloads are interpreted by the application layer.

## GenericProviderRuntimeHint

NDNSF core provider state summary reusable by non-DI applications.

Fields:
- `providerName`
- `timestampMs`
- `activeWorkCount`
- `queueLength`
- `estimatedQueueWaitMs`
- `capacityHints`
- `peerMetrics`
- `confidence`

Validation:
- Negative queue or work counts are invalid.
- Capacity hints are typed key/value metadata and must not require DI semantics.

## PeerNetworkMetric

NDNSF core directed provider-to-provider or provider-to-peer metric.

Fields:
- `srcPeer`
- `dstPeer`
- `rttMs`
- `bandwidthMbps`
- `lossRate`
- `jitterMs`
- `bytesSampled`
- `updatedAtMs`
- `confidence`

Validation:
- Metrics are directed.
- Loss rate must be between 0 and 1.
- Bandwidth must be positive when present.

## GenericAdmissionLease

NDNSF core short-lived provider admission reservation.

Fields:
- `leaseId`
- `requestId`
- `serviceName`
- `providerName`
- `status`
- `reasonCode`
- `estimatedStartMs`
- `estimatedFinishMs`
- `expiresAtMs`
- `resourceBindingSchema`
- `resourceBinding`

Validation:
- Valid leases require `leaseId`, `requestId`, `serviceName`, `providerName`,
  and `expiresAtMs`.
- The core stores and compares resource binding bytes or digest, but does not
  interpret service-specific payloads.

## GenericLeaseValidationResult

NDNSF core decision when consuming a selected lease.

Fields:
- `status`
- `reasonCode`
- `leaseId`
- `requestId`
- `serviceName`
- `providerName`

Reason examples:
- `LEASE_EXPIRED`
- `LEASE_NOT_FOUND`
- `LEASE_ALREADY_CONSUMED`
- `LEASE_REQUEST_MISMATCH`
- `LEASE_SERVICE_MISMATCH`
- `LEASE_BINDING_MISMATCH`
- `QUEUE_OVERLOADED`

## ModelFragmentKey

Canonical fragment identity.

Fields:
- `modelId`
- `modelVersion`
- `modelDigest`
- `runtimeBackend`
- `precision`
- `splitStrategy`
- `stageIndex`
- `stageCount`
- `layerStart`
- `layerEnd`
- `shardIndex`
- `shardCount`
- `fragmentDigest`

Validation:
- Digest fields are required when comparing fragments across providers.
- `stageIndex < stageCount`.
- `shardIndex < shardCount`.
- Layer range must be valid for the model spec.

## DiFragmentRuntimeState

Provider-reported state for one fragment.

Fields:
- `fragmentKey`
- `residency`
- `estimatedReadyMs`
- `pinned`
- `lastUsedMs`
- `memoryFootprintMb`
- `confidence`

Residency values:
- `GPU_LOADED`
- `CPU_RESIDENT`
- `DISK_RESIDENT`
- `REPO_AVAILABLE`
- `MISSING`

Validation:
- `estimatedReadyMs` is zero or near-zero for GPU-loaded fragments.
- Missing fragments must carry high ready cost.

## DiProviderRuntimeState

DI-specific provider dynamic runtime state used during planning. It is carried
inside `GenericAckMetadata.servicePayload`.

Fields:
- `providerName`
- `timestampMs`
- `activeRoleCount`
- `queueLength`
- `estimatedQueueWaitMs`
- `freeGpuMemoryMb`
- `freeCpuMemoryMb`
- `supportedBackends`
- `fragmentStates`
- `kvCacheHints`
- `confidence`

Validation:
- Timestamp must be present.
- Negative memory or queue values are invalid.
- Generic queue and peer metrics should be taken from `GenericProviderRuntimeHint`.

## DiLeaseResourceBinding

DI-specific resource payload carried by `GenericAdmissionLease`.

Fields:
- `roleId`
- `fragmentKey`
- `residency`
- `reservedGpuMemoryMb`
- `reservedCpuMemoryMb`
- `estimatedReadyMs`

Validation:
- Role id must match the selected DI role.
- Fragment key must match the offered DI fragment.
- The payload is opaque to NDNSF core except for byte/digest comparison.

## ProviderNetworkMatrix

DI planner view over generic directed peer metrics.

Fields:
- `metrics`
- `defaultRttMs`
- `defaultBandwidthMbps`
- `stalePenaltyMs`
- `unknownPenaltyMs`

Validation:
- Missing directed metrics must fall back conservatively.
- Stale or low-confidence metrics must be penalized.

## DiLeaseValidationResult

DI decision after core lease validation succeeds.

Fields:
- `status`
- `reasonCode`
- `leaseId`
- `requestId`
- `roleId`
- `fragmentKey`
- `providerName`

Reason examples:
- `LEASE_ROLE_MISMATCH`
- `LEASE_FRAGMENT_MISMATCH`
- `FRAGMENT_EVICTED`
- `INSUFFICIENT_GPU_MEMORY`

## ReplanRecord

User-side evidence for stale state and retries.

Fields:
- `requestId`
- `attempt`
- `failedProvider`
- `failedLeaseId`
- `reasonCode`
- `excludedProviders`
- `nextAction`

Validation:
- Attempt count must not exceed `maxReplanAttempts`.

## PlannerMetrics

Experiment and diagnostic output.

Fields:
- `requestId`
- `selectedAssignments`
- `scoreBreakdown`
- `leaseCounters`
- `residencyCounters`
- `edgeCostSummary`
- `queueWaitSummary`
- `replanCount`
- `latencyMs`
- `success`
- `failureReason`
