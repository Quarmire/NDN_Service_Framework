# Orchestration Migration Evidence

Repo network, SQLite, placement, catalog and repair orchestration moved from
`ndnsf_distributed_inference.repo` to
`py_repoclient.orchestration`. The DI default package no longer exports Repo
node/server/client policy. DI retains only `repo_reference.py`, which converts
artifact metadata to and from public Repo references.

No wire, SQLite schema or operation behavior changed in this ownership slice.

```text
exact packet tests       12/12 PASS
tiered cache tests       11/11 PASS
HA/restart/quorum tests  47/47 PASS
repair sidecar tests      2/2 PASS
campaign evidence tests   3/3 PASS
core envelope tests       7/7 PASS
DI boundary tests         8/8 PASS
```

The default import negative check confirms `ndnsf_distributed_inference` has no
`RepoNodeApp`, `DistributedRepo`, `NetworkDistributedRepoClient`, or
`RepoObjectManifest` export.
