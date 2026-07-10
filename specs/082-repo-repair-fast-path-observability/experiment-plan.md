# Experiment Plan

## Material Passport

- Experiment ID: `repo-repair-fast-path-082`
- Type: deterministic code/network campaign
- Verification status: verified
- Data: local MiniNDN logs and JSON/CSV summaries

## Research Question

Does removing a redundant target-miss preflight reduce bounded-window repair
latency and expose enough phase evidence to explain remaining delay?

## Variables

- Independent variable: target `FETCH_PREPARE` preflight present versus absent.
- Fixed: seed 78004, workers=3, max-jobs=6, 60 seconds, 0.5 RPS, concurrency 4,
  RF=3, W=QUORUM, 2 KiB objects, RepoA failure at 20 seconds and restart after
  12 seconds.
- Outcomes: outage repair count/coverage, first/last repair latency, cycle phase
  time, request p50/p95, success, receipt floor, invalid repairs.

## Interpretation Rules

- Correctness gates dominate performance: 30/30, W>=2, invalid repairs=0.
- Report the treatment once; do not rerun only to seek a favorable result.
- A single run is diagnostic evidence, not a production SLO.
- Attribute improvement only to the removed preflight and added visibility;
  do not generalize to larger objects or other topologies.
