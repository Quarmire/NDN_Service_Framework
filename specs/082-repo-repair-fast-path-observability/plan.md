# Implementation Plan: Repo Repair Fast Path and Observability

## Root Cause

Spec 081 workers=3 submitted three repair calls from worker threads, but every
NDNSF request ultimately entered one stable
`NetworkDistributedRepoClient._control_executor`. Each repair first sent
`FETCH_PREPARE` to a catalog-known missing target. The negative ACK produced a
selection timeout of roughly five seconds, serializing fixed delay before any
source transfer and causing multiple concurrent repairs to miss the campaign
window.

## Runtime Changes

1. Extend `_scan_repair_jobs()` with SQL-derived state and claimability counts.
2. Record peer delta/merge and repair-cycle structured metrics in the sidecar.
3. Remove the target `FETCH_PREPARE` preflight from `catalog_repair()`.
4. Preserve source preparation, exact Data retrieval, hashes, packet manifest,
   repair authorization, target persistence, durable lease, and completion.
5. Parse sidecar metrics into MiniNDN `summary.json`.

## Safety

The durable catalog plan is the proof that the target is missing for this
generation. Repair replay remains safe because the target receives the same
object name and digest through `STORE_PACKET_PULL`, validates the source packet
manifest and hashes, and commits only an authorized repair copy. A lost
completion response may cause repeated transfer work but not divergent data.

## Experiment

- Historical baseline: Spec 081 finalized workers=3, seed 78004.
- Treatment: identical topology, workload, failure/restart, seed, workers=3,
  max-jobs=6, with the target preflight removed.
- Primary: repaired strict outage objects, first/last repair latency.
- Secondary: repair-cycle phase time, claimable/claimed counts, request p50/p95,
  W receipt floor, invalid repair events.
- Accept a negative result if the source/target data path remains limiting.

## Validation

- Focused scan, fast-path, sidecar, and evidence tests.
- Full Repo Python and focused C++/Targeted/security/worker regressions.
- One 60-second matched MiniNDN treatment campaign.
- Spec Kit, CodeGraph, GSD, docs, and diff checks.

## Accepted Outcome

The single planned workers=3 treatment completed 30/30 requests with W=2 and
zero invalid repairs. Strict outage repair improved from 1/4 to 4/4, first
repair from 20.248 to 10.587 seconds, and request p95 from 5.661 to 1.814
seconds. Initial post-merge visibility exposed nine claimable jobs and drained
six in one bounded cycle. The target preflight was the confirmed bottleneck.
