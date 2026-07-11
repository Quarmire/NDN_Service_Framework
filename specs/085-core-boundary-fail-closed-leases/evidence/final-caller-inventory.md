# Final Caller Inventory

Date: 2026-07-10

Exact searches found:

- no `ExecutionArtifact*`, `RepoDataPlaneProducer`, `RetryPolicy`, or
  `retry_call` export under `pythonWrapper/ndnsf`;
- no `GRANTED_LOCAL`, coordinator lease acquire/release, or generic deployment
  eviction method in active runtime code;
- execution artifact symbols only under DI and its boundary tests;
- `RepoDataPlaneProducer` Python wrapper only under `py_repoclient`; the native
  binding remains in `_ndnsf` as the transport implementation;
- retry-by-error-string only under the DI package and DI callers;
- generic segmented/exact Data, status, coordination-envelope, telemetry, and
  Core execution-lease symbols remain intentionally in Core.

`tools/maintenance/ndnsf_occam_audit.py` still reports broader parent-084
findings: V1/BloomFilter removal belongs to Spec086, coordination review to
Spec087, stream to Spec089, and typed legacy status to Spec090. Occurrences in
the new DI and Repo target packages are intended ownership, not Core leakage.
