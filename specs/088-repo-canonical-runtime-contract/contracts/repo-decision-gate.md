# Repo Decision Gate

**ADR status**: APPROVED

Canonical object/local implementation: C++ `NDNSF-DistributedRepo` library.

Canonical deployed network runtime: bindings and versioned NDNSF operational
orchestration in `py_repoclient`. The former unversioned standalone C++ network
app is removed rather than maintained as a second runtime.

Public names represent object operations. Private replication and repair names
are versioned, permission-protected, and unavailable to ordinary clients.

Deletion gate: no DI-local Repo implementation is removed until exact packet,
restart, cache, quorum, tombstone, catalog, repair, malformed and authorization
fixtures pass through the migrated client.
