## Material Passport

- **Artifact type**: Experiment Result
- **Verification status**: VERIFIED
- **Date**: 2026-07-10
- **Environment**: MiniNDN AI_Lab, three persistent Repo providers
- **Measured window**: 60 seconds
- **Configuration**: 0.5 RPS, concurrency 4, 10% reads, 2,048-byte objects,
  RF=3, W=QUORUM, Targeted with bounded fallback, 5,000 ms timeout

# Spec 080 Results

## Availability And Latency

- Requests: 30/30 successful; achieved 0.4999 RPS.
- Successful writes: 27; minimum validated receipt count was 2.
- Overall latency p50/p95/p99: 169.558/2,024.927/5,603.045 ms.
- Before RepoA loss: 11/11 requests; nine writes had three receipts.
- After RepoA loss: 19/19 requests; 18 writes had exactly two receipts.
- Targeted control: 133 calls, one timeout, two bounded fallbacks, maximum four
  concurrent replica calls.

## Online Recovery

RepoA stopped at the 20-second failure epoch and restarted 12 seconds later.
The harness restarted RepoA's catalog sidecar in auto-repair mode with the same
identity, peers, policy, and persistent storage directory.

- Five successful writes completed strictly between failure and restart.
- The recovered sidecar created ten durable repair jobs.
- It completed three repairs before the 60-second workload ended.
- One completed repair belonged to the strict five-object outage set, so
  bounded-window outage repair coverage was 1/5 = 20%.
- The first recovered-target repair completed 15,015 ms after restart; the last
  observed repair completed 27,004 ms after restart.
- Four outage-window objects remained explicitly unrepaired at measurement end.

For repaired outage object
`/example/repo/user/NDNSF-DISTRIBUTED-REPO/OBJECT/CAMPAIGN/78004/15`, RepoA's
persistent database contained the object manifest and exact packet reference.
Its local catalog journal contained AVAILABLE entries from RepoA, RepoB, and
RepoC with the same object digest, proving restored RF=3 rather than a log-only
success marker.

## Interpretation And Limits

The experiment validates the full online path: quorum write during failure,
Repo and sidecar rejoin, catalog anti-entropy, durable job creation/claim, NDNSF
repair transfer, target persistence, and RF=3 catalog convergence. Repair runs
concurrently with continued requests and does not block quorum commit.

It does not show that every missing replica repairs within 60 seconds. In this
run each observed repair took roughly six seconds of wall time, so the short
post-restart interval completed only part of the backlog. This is accepted as a
bounded-window result and identifies repair throughput/fair scheduling as the
next performance boundary.

## Result Path

`results/repo_targeted_spec080_rf3_quorum_recovery_20260710/campaign-c4-rps0.5-seed78004`
