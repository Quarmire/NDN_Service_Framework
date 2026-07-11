# MiniNDN RF=2 / W=ALL Campaign Evidence

Date: 2026-07-11

Matched treatment: AI Lab topology, exact Data packets, RF=2, W=ALL, 60-second
measured window, 0.5 offered RPS, concurrency 4, 90% reads, 4096-byte objects,
Targeted control path, 30-second request deadline. Only the seed changed.

| Run | Seed | Success | Achieved RPS | p50 ms | p95 ms | Min write receipts |
|---|---:|---:|---:|---:|---:|---:|
| pass-1 | 88201 | 30/30 | 0.49998 | 141.129 | 5196.588 | 2 |
| pass-2 | 88202 | 30/30 | 0.49997 | 136.607 | 5166.054 | 2 |
| pass-3 | 88203 | 30/30 | 0.49989 | 129.971 | 5254.333 | no writes selected |

Raw results are under `results/spec088-rf2-wall-20260711/pass-{1,2,3}`.
All finalized writes reported two successful receipts and no request rejection.

An initial concurrency-1 pilot is retained at `run-1`: 26/30 completed and four
requests were intentionally rejected by client admission while two cold reads
occupied the sole slot for about five seconds. It is diagnostic evidence, not
counted as a passing matched run.

After correcting the catalog sidecar to use versioned operation services, an
additional 60-second stop/restart run completed 12/12 requests and 16 repair
scan cycles with two successful catalog merges (one inline, one segmented
pull), zero invalid repair events, and no claimable repair because persistent
repoA retained its RF=2 data across restart. Raw evidence is under
`results/spec088-rf2-wall-20260711/repair-validation`.
