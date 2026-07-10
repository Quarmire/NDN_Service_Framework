# Results: Repo Repair Fast Path and Observability

## Correctness Gates

- Python compile and 78 Repo Python tests passed.
- Four C++ Repo checks passed.
- Targeted 14, crypto/auth 6, and worker 10 tests passed.
- MiniNDN completed 30/30 requests with receipt floor W=2.
- `invalidRepairEventCount=0` and no failed-only write was repaired.

## Matched Workers=3 Result

| Metric | Spec 081 preflight | Spec 082 fast path |
|---|---:|---:|
| Request success | 30/30 | 30/30 |
| Request p50 | 318.392 ms | 254.313 ms |
| Request p95 | 5,660.957 ms | 1,814.117 ms |
| Strict outage repair | 1/4 (25%) | 4/4 (100%) |
| First repair after restart | 20.248 s | 10.587 s |
| Recovered-target repairs | 1 | 18 |
| Repair throughput | 0.0384/s | 0.6915/s |

The initial post-merge cycle reported ten new jobs, nine claimable jobs, six
claims, and six completions. Those six transfers completed in 0.838 seconds;
individual transfer duration was roughly 0.30--0.47 seconds instead of the old
fixed 5--10 second path. Across the recovery window, 12 cycles claimed and
completed 18 jobs with no repair failure.

## Interpretation

The target `FETCH_PREPARE` was a redundant known-miss probe. Its negative ACK
waited for selection timeout on the single client owner thread, serializing
worker startup. Removing it exposed real bounded transfer concurrency while
retaining source prepare, exact Data retrieval, packet and object hashes,
repair authorization, target persistence, durable leases, and completion.

The next measured bottleneck is catalog merge: 12 recovered-sidecar merge
events consumed 5.200 seconds, and the initial two peer deltas required 16
batches each. This single campaign is strong diagnostic evidence but not a
production recovery SLO.
