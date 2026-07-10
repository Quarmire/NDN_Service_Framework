# Research Decisions

## Provider Authority Instead Of Global Coordination

Each provider is authoritative for its local finite resources. User-side
prepare/commit/abort supplies distributed atomicity. A coordinator may advise
future planning but is unnecessary for correctness and cannot grant leases.

## Boot Epoch Instead Of Lease Persistence

Active execution leases are volatile. Every provider boot generates a new
epoch; delayed operations from an old boot are rejected. Users replan after a
restart. This is simpler and safer than pretending an interrupted execution
continues from durable lease state.

## Application Service Instead Of New Core Wire Protocol

DI lease operations use the existing V2 dynamic API and security path. Core
owns only application-neutral envelopes and the provider-local table. This
avoids adding DI operation names to Core C++ wire parsing.

## Separate Admission And Execution Leases

The existing GenericAdmissionLease is a one-shot ACK/Selection reservation and
is consumed before request execution. Execution leases have prepare/commit,
long-running pinning, renewal, release, restart epoch, and hard-deadline
semantics. Reusing one state machine would blur two lifecycles; they share
identity/binding conventions but remain separate tables.

## Opaque Binding Proof In Core

Core C++ compares opaque canonical bytes and does not parse DI role/fragment
JSON. DI serializes its resource binding deterministically and supplies the
proof, preserving the Core/application boundary.

Core additionally compares opaque `conflictKeys` to make prepare atomic. DI
maps its provider-local worker/GPU slot assignment to keys; Core does not know
what those keys mean. This prevents distinct fragment bindings from silently
claiming the same exclusive compute slot.

## No Global RefCount

Global refCount is stale under partition and creates a hidden authority.
Eviction is local: each provider checks active committed leases that bind its
local fragment/resource. Deployment records remain descriptive.

## Repo Move Is Ownership-Only

Moving `RepoDataPlaneProducer` changes import ownership but not packet names,
storage, SQLite, catalog, repair, or canonical runtime. Those remain Spec 088.
