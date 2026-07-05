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

## FragmentRuntimeState

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

## ProviderRuntimeState

Provider dynamic runtime state used during planning.

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
- `peerMetrics`
- `confidence`

Validation:
- Timestamp must be present.
- Negative memory or queue values are invalid.
- Peer metrics are directed.

## ProviderPairMetric

Directed provider-to-provider network metric.

Fields:
- `srcProvider`
- `dstProvider`
- `rttMs`
- `bandwidthMbps`
- `lossRate`
- `jitterMs`
- `bytesSampled`
- `updatedAtMs`
- `confidence`

Validation:
- `srcProvider != dstProvider` unless local-edge optimization is explicitly represented.
- Bandwidth must be positive when present.
- Loss rate must be between 0 and 1.

## LeaseOffer

Short-lived provider admission reservation.

Fields:
- `leaseId`
- `requestId`
- `providerName`
- `roleId`
- `fragmentKey`
- `status`
- `reasonCode`
- `reservedGpuMemoryMb`
- `reservedQueueSlot`
- `estimatedStartMs`
- `estimatedFinishMs`
- `expiresAtMs`

Validation:
- Valid lease must include `leaseId`, `requestId`, `roleId`, `fragmentKey`, and `expiresAtMs`.
- Rejected offers must include a reason code.

## LeaseValidationResult

Provider decision when consuming a selected lease.

Fields:
- `status`
- `reasonCode`
- `leaseId`
- `requestId`
- `roleId`
- `fragmentKey`
- `providerName`

Reason examples:
- `LEASE_EXPIRED`
- `LEASE_NOT_FOUND`
- `LEASE_ALREADY_CONSUMED`
- `LEASE_ROLE_MISMATCH`
- `LEASE_FRAGMENT_MISMATCH`
- `FRAGMENT_EVICTED`
- `QUEUE_OVERLOADED`
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
