# Data Model

## GenericExecutionLease

| Field | Rule |
|---|---|
| `schema` | `ndnsf-execution-lease-v1` |
| `leaseId` | provider-issued non-empty identity |
| `providerName` | authoritative provider |
| `providerEpoch` | changes on every provider-table construction |
| `requesterName` | authenticated requester binding |
| `requestId` | logical DI transaction identity, scoped by authenticated requester |
| `serviceName` | canonical unified service name |
| `planDigest` | immutable plan-content digest |
| `resourceBindingSchema` | application-neutral schema label |
| `resourceBindingProof` | canonical opaque bytes compared by Core; DI may derive them from stable JSON |
| `conflictKeys` | opaque application-supplied resource keys; overlap with any active lease is rejected |
| `state` | PREPARED, COMMITTED, EXECUTING, ABORTED, RELEASED, EXPIRED |
| `expiresAtMs` | bounded cleanup time |
| `executionDeadlineMs` | hard cleanup deadline after activation |
| `idempotencyKey` | duplicate-operation identity |

Each PREPARE/COMMIT/ABORT/RENEW/RELEASE is a separate NDNSF invocation and has
its own authenticated wire request ID. That wire ID is validated as present by
the service path but is not reused as the logical transaction ID because doing
so would conflict with NDNSF replay protection. The payload transaction ID is
integrity-protected by NDNSF and is always evaluated together with the
authenticated requester identity; Core rechecks requester ownership on every
state-changing operation.

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
