# Entry Inventory

- C++ `RepoCore/RepoNode/RepoClient/RepoProtocol` validates exact Data names,
  immutable wire conflicts, manifests and object operations.
- Deployed `DistributedRepoNodeApp` selects SQLite or tiered SQLite+LRU; cache
  budget zero disables memory admission without changing authority.
- `py_repoclient` exposes the C++ contract and native Data producer.
- `py_repoclient.orchestration` now owns network orchestration, placement,
  catalog and repair. DI contains only lightweight reference conversion.
- The C++ library is the object/local-service authority. The duplicate
  unversioned standalone C++ network app is removed; the Python adapter is the
  sole deployed NDNSF Repo runtime and uses versioned public/internal services.
- Tests cover exact packets, tiered cache, HA, repair, authorization, runtime
  boundaries and matched MiniNDN campaigns.
