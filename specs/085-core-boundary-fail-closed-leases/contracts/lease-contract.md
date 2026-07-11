# Execution Lease Contract

## Operations

| Operation | Valid source state | Success state |
|---|---|---|
| PREPARE | absent or identical PREPARED replay | PREPARED |
| COMMIT | PREPARED or identical COMMITTED replay | COMMITTED |
| VALIDATE_AND_ACTIVATE (provider-local) | COMMITTED or identical EXECUTING replay | EXECUTING |
| ABORT | PREPARED or COMMITTED | ABORTED |
| RENEW | PREPARED, COMMITTED, or EXECUTING and before hard deadline | unchanged with later expiry |
| RELEASE | EXECUTING, COMMITTED, or identical terminal replay | RELEASED |
| VALIDATE | COMMITTED and unexpired | COMMITTED |

## Required Reasons

`LEASE_UNAVAILABLE`, `LEASE_NOT_FOUND`, `LEASE_EXPIRED`, `LEASE_STALE_EPOCH`,
`LEASE_IDEMPOTENCY_CONFLICT`, `LEASE_INVALID_TRANSITION`,
`LEASE_REQUESTER_MISMATCH`, `LEASE_REQUEST_MISMATCH`,
`LEASE_SERVICE_MISMATCH`, `LEASE_PLAN_MISMATCH`, `LEASE_BINDING_MISMATCH`,
`LEASE_CAPACITY_REJECTED`, and `LEASE_INTERNAL_ERROR`.

`VALIDATE_AND_ACTIVATE` is an atomic validate-and-transition operation called by the provider
execution handler immediately before business logic. Repeated roles on the same
provider replay the transaction activation idempotently. The user-owned
transaction releases every provider lease in a finally-style completion path
after the whole collaboration. PREPARED/COMMITTED use ordinary TTL;
EXECUTING remains pinned until release or its separate hard deadline.

Only PREPARE, COMMIT, ABORT, RENEW, and RELEASE are exposed through the DI lease
service. Validation/activation is internal to the actual execution handler.

Every network operation has a distinct NDNSF wire request ID. The DI payload's
`requestId` is therefore a logical transaction ID, not a replacement wire ID.
The service accepts identity/provider/service and the per-operation wire ID only
from authenticated context. Core binds the logical transaction ID to that
requester and rejects commit/abort/renew/release from any other requester.

## Atomicity

The user persists transaction state in memory for the current invocation:
selected providers, prepare results, commit results, and cleanup status.
Execution begins only after all commits succeed. Any incomplete transaction is
non-executable and is cleaned through abort/release plus provider TTL.

Prepare atomically rejects any `conflictKey` already held by a nonterminal,
unexpired PREPARED, COMMITTED, or EXECUTING lease. DI must supply at least one
provider-issued key for an exclusive role. A requester cannot choose the final
keys; the provider service derives them from trusted local inventory and returns
them in the lease. Terminal/expired leases release their keys. Core treats keys
as opaque strings.

## Observability

Counters: prepared, committed, activated, aborted, released, expired, renewed, rejected by
reason, idempotent replay, conflict, stale epoch, cleanup timeout, and active
prepared/committed/executing gauges. No counter may contain secrets or artifact payloads.
