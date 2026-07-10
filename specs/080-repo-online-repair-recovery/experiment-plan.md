## Material Passport

- **Schema**: ARS-9
- **Artifact type**: Code experiment plan
- **Experiment ID**: repo-online-repair-recovery-080
- **Status**: Completed
- **Environment**: Local MiniNDN AI_Lab topology
- **Data access**: Generated workload and local logs only
- **Result artifact**: `specs/080-repo-online-repair-recovery/results.md`

# Experiment Plan

## Research Question

Can NDNSF-REPO preserve RF=3/W=QUORUM write availability during one Repo node
failure and automatically restore outage-window objects to RF=3 after that node
restarts?

## Hypothesis

Writes will commit with two validated receipts while RepoA is offline. Once
RepoA and its catalog sidecar rejoin, the durable repair planner will select
RepoA for objects written during the outage and complete at least one validated
repair within the same 60-second campaign.

## Variables

- Independent variable: RepoA availability state: healthy, offline, recovered.
- Primary dependent variables: successful writes, confirmed receipts, repaired
  outage objects, repair coverage, repair completion latency.
- Secondary dependent variables: achieved RPS, p50/p95 operation latency,
  Targeted timeout/fallback counts, repair retries/failures.
- Controls: topology, seed, RF/W, offered load, concurrency, object size,
  request timeout, failure epoch, restart delay, and measured duration.

## Procedure

1. Start Controller, three Repo providers, and catalog sidecars.
2. Complete bounded readiness/seed setup outside measurement counters.
3. Start the 60-second workload.
4. Stop RepoA at +20 seconds and permit quorum writes on RepoB/RepoC.
5. Restart RepoA and its auto-repair sidecar 12 seconds later.
6. Continue workload and repair until the measured window ends.
7. Correlate successful outage-window write object names with repair events
   whose target is RepoA.
8. Preserve raw lifecycle CSV, node logs, catalog logs, and summary JSON.

## Acceptance And Interpretation

- Required: at least one successful outage-window write has exactly two valid
  receipts; at least one such object later has a successful RepoA repair event.
- Required: no successful write has fewer than two receipts.
- Report repair coverage as a descriptive campaign result, not a universal
  probability claim.
- A partial repair set is a bounded-window result, not hidden or retried until
  it appears perfect.
- A failure before fault injection remains classified separately and is not
  attributed to RepoA loss.

## Reproducibility

The final quickstart records the exact command, seed, result directory,
software revision, environment, and acceptance checks. The run has process
timeouts and preserves anomalies rather than silently rerunning them.
