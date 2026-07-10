# Results: Bounded Parallel Replica Repair

## Correctness

- Build and Python compile passed.
- 75 Repo Python tests passed.
- Four C++ Repo checks passed.
- Targeted 14, crypto/auth 6, and worker 10 tests passed.
- Both matched MiniNDN campaigns completed 30/30 requests.
- Every successful write had at least W=2 validated receipts.
- Both campaigns reported `invalidRepairEventCount=0`.

Multi-replica writes remain `STAGED` until receipt-backed `FINALIZE_WRITE`.
Staged generations are not readable, advertised, or repairable. This prevents
partial writes below quorum from being resurrected by recovery repair.

## Matched Campaigns

| Workers | Success | Request p50/p95 | Strict outage repair | First repair | Repair throughput |
|---:|---:|---:|---:|---:|---:|
| 1 | 30/30 | 239.371/5,371.381 ms | 2/4 (50%) | 16.222 s | 0.0959/s |
| 3 | 30/30 | 318.392/5,660.957 ms | 1/4 (25%) | 20.248 s | 0.0384/s |

## Interpretation

The bounded worker implementation is correct, and synthetic independent-job
tests prove that transfers overlap without concurrent `ServiceUser` control
calls. The 60-second network treatment did not improve backlog drain. Only one
repair became executable on the recovered target in the workers=3 run, so the
bottleneck was catalog/control-path job visibility rather than transfer worker
capacity. The production default therefore remains one worker. The configurable
pool is retained for deployments where multiple independent jobs are already
claimable, but no performance claim is made from this campaign.

These are single matched runs and are not a production recovery SLO.
