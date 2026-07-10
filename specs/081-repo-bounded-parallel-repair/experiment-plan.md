## Material Passport

- **Schema**: ARS-9
- **Artifact type**: Code experiment plan
- **Experiment ID**: repo-bounded-parallel-repair-081
- **Status**: Planned
- **Environment**: MiniNDN AI_Lab
- **Data access**: Generated workload and local logs only

# Experiment Plan

## Research Question

Does bounded three-worker repair drain more of a recovered Repo's backlog
within 60 seconds than the accepted single-worker path without reducing quorum
write availability?

## Variables

- Independent variable: repair workers, 1 versus 3.
- Fixed controls: topology, seed 78004, 0.5 RPS, concurrency 4, 10% reads,
  2,048-byte objects, RF=3, W=QUORUM, 5,000 ms timeout, failure at 20 seconds,
  restart after 12 seconds.
- Primary outcomes: strict outage repaired-object count and coverage.
- Secondary outcomes: total repair completions, completion latency, request
  success, achieved RPS, write latency, receipts, Targeted failures/fallbacks.

## Interpretation

This is a matched deterministic campaign comparison. A larger repaired set is
evidence that transfer serialization was a bottleneck in this setup. Equal or
worse coverage is retained as evidence that control/data contention dominates.
No significance test or production SLO claim is made from one pair.
