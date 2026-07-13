# Spec 110 Source Baseline

Captured before the first Spec 110 implementation artifact was added. The
worktree was intentionally sealed dirty because the Spec 109 erratum and the
audited Spec 110 design were not yet committed; a clean-commit claim would be
false.

| Field | Value |
|---|---|
| Captured at | `2026-07-13T18:00:00Z` |
| Branch | `Experimental` |
| HEAD | `8b9a4fe709d35b9e4d4961eaa25cefad45cfc0b2` |
| Worktree | `SEALED_DIRTY` |
| Source snapshot | `sha256:f953765aa74270555c54ac9ada60bdc07a1fbb5326072eefd2c1b2e1e5d066c0` |
| HEAD tree | `sha256:fd7199c8221ce1e8e6d57d4d3855e2c7333241a450a40d5a2f2d637deac30fb5` |
| Binary diff | `sha256:0204118a3cb973c777c2ab9ee51a89c030c659cf8003409e0615391b187d8ce5` |
| Untracked manifest | `sha256:b267097a273670ebfeb2c520eb65300689a8459e1fba5f5e5c0c7ef1a12968b6` |
| Untracked archive | `sha256:208790b2db80fd5a1a87798c3f44ff8ecbedd33b3382baa5ff931b30c4c4e954` |
| Spec 109 erratum | `sha256:96ac377a9f8becd94e0c0e0809f76f0544519258adfd8f39d8d7400fd4ea2015` |

The snapshot was produced with `spec109_source.capture_source_snapshot()` and a
temporary deterministic archive at `/tmp/spec110-source-untracked.tar`. The
archive is a local reconstruction aid, not durable experiment evidence and not
an allowed source of credentials or model data. The campaign binds the digest,
not that temporary path.

CodeGraph was current at capture: 7,295 files, 160,816 nodes, 181,676 edges,
464.74 MB, built-in SQLite/WAL backend. Later implementation changes must create
a new source binding; they do not mutate this baseline.
