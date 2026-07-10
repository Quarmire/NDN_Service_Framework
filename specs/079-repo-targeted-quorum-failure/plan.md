# Implementation Plan: Targeted Quorum Provider Failure

## Architecture

1. Pass `required_acks` into reservation coordination.
2. Accept partial reservations only when their validated count reaches W.
3. Store only to successfully reserved providers; keep the manifest's desired
   replication factor unchanged and record only durable receipt owners.
4. Feed Targeted outcomes into existing provider health/cooldown tracking and
   omit active-cooldown explicit replicas when W can still be met. Treat a
   failed Normal fallback as stronger unavailability evidence than a Targeted
   timeout alone.
5. Timestamp every campaign row and have the MiniNDN harness classify rows
   relative to the injected failure epoch.
6. Run matched no-failure and RepoA-failure campaigns at RF=3/W=QUORUM.

## Safety Invariants

- W is a receipt threshold, never a fabricated-replica shortcut.
- A reservation response is not a write receipt.
- Fewer than W reservations or receipts fails the operation.
- W=ALL behavior remains unchanged.
- Missing desired replicas are visible through replicationFactor versus
  confirmedReplicaNodes and remain repair work.

## Experiment Variables

- Topology: MiniNDN AI_Lab, three Repo providers.
- Workload: 60 seconds, 0.5 RPS, concurrency 4, 10% reads, 2,048-byte objects.
- Storage policy: RF=3, W=QUORUM, Targeted control, bounded fallback enabled.
- Request timeout: 5,000 ms.
- Seed: 77903.
- Treatment: RepoA stopped 20 seconds after the ready barrier; no restart.

## Validation

- Focused Python quorum/reservation/cooldown tests.
- Existing 61+ Repo Python tests and four C++ Repo checks.
- Targeted C++/Python and NAC-ABE concurrency regressions.
- Two matched 60-second MiniNDN campaigns.
