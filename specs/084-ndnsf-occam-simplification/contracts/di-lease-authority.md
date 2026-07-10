# DI Lease Authority Contract

## Authority Model

There is no global execution-lease or deployment-refcount authority. Each
provider is authoritative only for its own finite resources and persists or
reconstructs enough local lease state to reject conflicting work.

- The user-side planner proposes a plan; it never authorizes provider capacity.
- A DI deployment record is descriptive and user-owned. It is not evidence that
  any provider has reserved resources.
- Every selected provider owns the admission and execution leases for the roles
  assigned to that provider.
- Advisory coordination may rank plans but cannot grant, commit, renew, release,
  or override a provider lease.
- An unavailable authority causes a typed `UNAVAILABLE` or `REJECTED` result.
  No local, untracked, or synthetic success lease is allowed.

## Distributed State Machine

```text
PROPOSED
  -> PREPARING
      -> PREPARED on every selected provider
          -> COMMITTING
              -> COMMITTED on every selected provider
                  -> EXECUTING -> RELEASED
      -> ABORTING -> ABORTED

Any reject, timeout, stale epoch, or provider restart before full COMMITTED
causes bounded abort of every prepared provider followed by bounded replanning.
```

Each request carries `planId`, `planDigest`, `requestId`, `provider`, `roleIds`,
`leaseId`, `leaseEpoch`, `state`, `expiresAt`, and an idempotency key. Prepare,
commit, abort, renew, and release are idempotent for the same key. A conflicting
digest or epoch is rejected.

## Failure And Recovery Rules

- A user starts execution only after every provider returns COMMITTED for the
  same plan digest and epoch.
- Prepare leases have a short TTL and consume bounded tentative capacity.
- Committed execution leases expire unless renewed; release is best effort and
  expiry is the final cleanup mechanism.
- A provider restart increments its lease epoch. Leases from an older epoch are
  invalid, and the user must replan or reacquire them.
- Provider-local eviction is permitted only when that provider has no active
  committed execution lease for the fragment. A global `refCount` is not used.
- Partial commit is resolved by abort/release on reachable providers and TTL on
  unreachable providers. It never becomes executable.
- Publication or discovery of a deployment record does not change lease state.

## Required Evidence

Tests must cover concurrent users, duplicate messages, delayed commit, partial
prepare, partial commit, authority timeout, stale epoch, restart, lease expiry,
renewal, release loss, and eviction during active execution. A MiniNDN campaign
must demonstrate zero conflicting committed role assignments without advisory
coordination.
