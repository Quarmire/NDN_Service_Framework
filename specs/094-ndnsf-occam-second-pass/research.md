# Occam Disposition Matrix

| Candidate | Decision | Evidence and replacement |
|---|---|---|
| DI process-pool open-loop driver | REMOVE | Spec 091/092 measured multi-second schedule slip from worker bootstrap; threaded passed the same gates through 8 RPS. |
| Old runtime-aware RPS sweep | REMOVE | Uses deterministic runner and success-only stability; canonical NativeTracer harness records schedule, throughput, dependency, and malformed traces. |
| Legacy DI GUI role profile/tabs | REMOVE | Current version-2 profile and direct USER/PROVIDER/CONTROLLER tabs provide the same role lifecycle with more complete configuration. |
| DI `repo_manifests` API alias | REMOVE | Exact scan finds no caller; `artifact_references` is the current name and semantic model. |
| Repo memory-only authoritative store | REMOVE | Production contract is SQLite authority plus bounded memory cache; memory store is used only by smoke/cache tests and default convenience constructors. |
| Repo `producer_retention_s` | REMOVE | Constructor explicitly discards it after always-on data plane migration. |
| Repo `isolated_runtime` | REMOVE | Helper explicitly discards it because duplicate ServiceUser/SVS identity is unsafe. |
| Nested `legacyStatus` metadata | REMOVE | Typed `ServiceOperationStatus.state/message/reason_code` is authoritative; no consumer reads the duplicate. |
| Repo capability service payload | KEEP | Domain-specific capacity/cache/inventory values are correctly nested under generic `ProviderCapabilityHint`. |
| Core mixed ACK reader | DEFER | Bounded migration contract: next major release or 2026-12-31, with explicit mode and counters. |
| Core stream C++ state + Python wrappers | KEEP | Python is a thin contract/binding layer; C++ is the single algorithm implementation. |
| UAV H264/FEC/ROI and mission logic | KEEP | Application policy with no Core duplicate; stream substrate remains Core-owned. |
| Repo catalog/replication/repair/reservations | KEEP | Required for distributed persistence and node-loss tolerance, not optional convenience. |
| DI leases/planning/cache/long context/runtime inventory | KEEP | Each prevents a concrete multi-user/resource/runtime failure and has active callers/tests. |
| Internal Repo native data-plane binding | DEFER | Not publicly exported; accepted internal binding with existing ownership review by 2026-12-31. |
| Large source-file splits | DEFER | Maintainability refactor, not mechanism deletion; separate feature if evidence justifies it. |
