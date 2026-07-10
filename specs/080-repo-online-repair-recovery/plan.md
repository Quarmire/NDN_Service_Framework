# Implementation Plan: Online Replica Repair After Recovery

## Architecture

1. Keep the Spec 079 RF=3/W=QUORUM write path unchanged: two durable receipts
   may commit, while `replicationFactor=3` remains repair intent.
2. Reuse the existing durable global catalog and `repair_jobs` state machine.
   A summary computes live owners, desired RF, fresh eligible targets, and one
   `RepoRepairAction` per missing replica.
3. Treat Repo process restart and catalog sidecar restart as one recovery
   operation in the MiniNDN harness. The sidecar rejoins with its existing
   persistent state and resumes peer merge, membership, scan, claim, repair,
   complete/fail cycles.
4. Reuse `NetworkDistributedRepoClient.catalog_repair`; do not add a second
   transfer protocol. Repair remains exact-name and hash validated.
5. Add object-level campaign correlation. Lifecycle CSV names each object;
   repair logs identify source, target, object, and completion epoch; the
   harness computes coverage over objects committed while the target was down.

## Recovery State Machine

```text
HEALTHY_RF3
  -- RepoA offline --> DEGRADED_RF2
  -- W=QUORUM writes --> DEGRADED_OBJECT_COMMITTED
  -- RepoA + sidecar restart --> TARGET_REJOINING
  -- fresh membership + catalog merge --> REPAIR_ELIGIBLE
  -- durable job claim --> REPAIRING
  -- validated target store --> RESTORED_RF3
  -- timeout/error --> RETRY_BACKOFF
```

## Safety Invariants

- Quorum availability never implies three confirmed replicas.
- Repair does not alter the original exact Data names or signed wire bytes.
- A stale target is not repair eligible.
- A leased job has at most one active owner; retries are idempotent.
- Recovered-target evidence is emitted only after target storage succeeds.
- Experiment instrumentation observes behavior but does not perform repair.

## Experiment Variables

- Topology: MiniNDN AI_Lab with RepoA, RepoB, RepoC.
- Workload: 60-second measured window, 0.5 RPS, concurrency 4, 10% reads,
  2,048-byte objects.
- Storage policy: RF=3, W=QUORUM, Targeted control, request timeout 5,000 ms.
- Fault: stop RepoA 20 seconds after ready; restart after 12 seconds.
- Recovery: auto-repair sidecars, 2-second scan interval, bounded job batch.
- Seed: 78004.

## Validation

- Focused catalog planner and durable repair-job unit tests.
- Full Repo Python suite and existing C++ Repo checks.
- Targeted and NAC-ABE focused regressions.
- One canonical 60-second MiniNDN outage/restart/repair campaign.
- Honest reporting of partial coverage, timeouts, and unrepaired objects.
