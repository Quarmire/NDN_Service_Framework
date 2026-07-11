# Repo Service Names And Authorization

Public object services use:

```text
/NDNSF/DistributedRepo/Object/v1/INSERT
/NDNSF/DistributedRepo/Object/v1/MANIFEST
/NDNSF/DistributedRepo/Object/v1/FETCH
/NDNSF/DistributedRepo/Object/v1/INVENTORY
/NDNSF/DistributedRepo/Object/v1/STATUS
/NDNSF/DistributedRepo/Object/v1/DELETE
/NDNSF/DistributedRepo/Object/v1/CATALOG_QUERY
```

Peer-only mutation services use:

```text
/NDNSF/DistributedRepo/Internal/v1/RESERVE_CAPACITY
/NDNSF/DistributedRepo/Internal/v1/RELEASE_CAPACITY
/NDNSF/DistributedRepo/Internal/v1/REPLICA_COMMIT
/NDNSF/DistributedRepo/Internal/v1/CATALOG_MERGE
/NDNSF/DistributedRepo/Internal/v1/CATALOG_DIGEST
/NDNSF/DistributedRepo/Internal/v1/REPAIR
```

Internal services require a Repo-peer NAC-ABE service attribute in addition to
normal one-time token and replay checks. Possession of ordinary object-client
permission never authorizes an Internal service. Unknown version/operation,
operation-in-payload attempts against the object root, and malformed payloads
fail closed with typed operation status.

Client-side write-transaction steps such as capacity preflight, packet commit,
and write finalization remain suboperations of the public versioned `INSERT`
service. They are authorized by publisher ownership and the selected write
transaction; they do not grant access to the peer namespace. The similarly
named Internal services accept only explicit `PEER_RESERVE_CAPACITY`,
`PEER_RELEASE_CAPACITY`, and `PEER_REPLICA_COMMIT` operations from authenticated
Repo provider identities. This distinction keeps the current user-coordinated
write protocol usable without giving an ordinary object client Repo-peer
authority.

During migration, old operation-in-payload names may be accepted only behind a
bounded compatibility flag with counters and an exit deadline. New producers
emit versioned names only.
