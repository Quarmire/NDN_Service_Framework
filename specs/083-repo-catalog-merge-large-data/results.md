# Results: Repo Catalog Merge Large-Data Path

## Correctness Gates

- Python compile and 80 Repo Python tests passed.
- The full build and focused Repo, Targeted, crypto/auth, and worker regressions
  passed. Two C++ checks initially contended for the same PIB when launched in
  parallel; both passed when rerun serially.
- MiniNDN completed 30/30 requests with receipt floor W=2.
- `invalidRepairEventCount=0`; all 4/4 strict-outage objects were repaired.
- The recovered sidecar used segmented pull for all six large deltas and inline
  merge for two small deltas, with no fallback.

## Matched Workers=3 Result

| Metric | Spec 082 batched merge | Spec 083 segmented pull |
|---|---:|---:|
| Request success | 30/30 | 30/30 |
| Request p50 | 254.313 ms | 208.015 ms |
| Request p95 | 1,814.117 ms | 1,779.222 ms |
| Strict outage repair | 4/4 | 4/4 |
| First repair after restart | 10.587 s | 9.033 s |
| Recovered merge events | 12 | 8 |
| Recovered merge batches | not parsed | 8 |
| Recovered merge time | 5,200.463 ms | 3,038.567 ms |
| Merge modes | batched inline | 6 pull, 2 inline |
| Segmented merge payload | not parsed | 186,260 bytes / 33 segments |

The first recovered peer deltas contained 37 and 39 catalog entries. They used
one protected pull each, transferring 70,407 bytes in 12 segments and 74,034
bytes in 13 segments. Both reported `fallback=0`.

## Interpretation

Large catalog deltas no longer consume one authenticated service invocation per
small entry batch. A compact control request names one immutable segmented
object and binds its schema, byte count, entry count, and SHA-256 digest. The
receiver fetches that exact name, validates the complete object, and only then
merges entries. Small deltas remain inline, and a bounded legacy batch fallback
keeps recovery available if the pull fails.

Against the fixed Spec 082 run, recovered-sidecar merge time fell by 41.6% and
first repair after restart improved by 1.554 seconds. Request p95 was effectively
stable. This is one deterministic diagnostic treatment, not a production
recovery SLO. The remaining first-repair delay is now dominated by restart,
sidecar/sync readiness, and the first peer merge cycle rather than serial merge
batches alone.
