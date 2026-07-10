# Data Model

## GenericExecutionLease

| Field | Rule |
|---|---|
| `schema` | `ndnsf-execution-lease-v1` |
| `leaseId` | provider-issued non-empty identity |
| `providerName` | authoritative provider |
| `providerEpoch` | changes on every provider-table construction |
| `requesterName` | authenticated requester binding |
| `requestId` | invocation identity |
| `serviceName` | canonical unified service name |
| `planDigest` | immutable plan-content digest |
| `resourceBindingSchema` | application-neutral schema label |
| `resourceBindingProof` | canonical opaque bytes compared by Core; DI may derive them from stable JSON |
| `conflictKeys` | opaque application-supplied resource keys; overlap with any active lease is rejected |
| `state` | PREPARED, COMMITTED, EXECUTING, ABORTED, RELEASED, EXPIRED |
| `expiresAtMs` | bounded cleanup time |
| `executionDeadlineMs` | hard cleanup deadline after activation |
| `idempotencyKey` | duplicate-operation identity |

For current one-worker providers, DI uses a key such as
`provider:<identity>:compute-slot:0`. A provider with multiple worker/GPU slots
assigns concrete slot keys during prepare. Keys are provider-issued; requester
keys are hints at most and are never authoritative. Core compares keys but does not
interpret model, fragment, GPU, or role semantics.

## LeaseOperationResult

Contains success, operation, lease state, reason code, provider epoch, lease ID,
expiry, and optional retry-after. Reason codes are enumerated in
`contracts/lease-contract.md`.

## DeploymentRecord

Contains plan ID/digest, creator, service, fragment/role map, artifact references,
and descriptive readiness. It contains no authoritative global refCount. A
provider's lease table is the only source for local pin/eviction decisions.

## BoundaryMigration

Tracks old symbol/import, target package, repository callers, external ABI
decision, migration commit, deletion commit, tests, and rollback command.
