# Frozen Performance Thresholds

These thresholds were recorded before child treatment code.

## Common Method

- Same topology, profiles, load, duration, warmup, seed set, NFD/SVS logging,
  runtime environment, and sample-rate configuration for baseline/treatment.
- At least three independent 60-second measured runs per ordinary comparison.
- Advisory-retention research uses at least ten paired runs, one predeclared
  primary metric, practical effect at least 10%, and paired 95% bootstrap CI
  excluding zero.
- Preserve raw per-run completion, failure reasons, p50, p95, throughput, CPU,
  memory, exact commands/environment, and canonical result paths.

## Blocking Limits

- All correctness/security tests pass; zero authorization bypass, conflicting
  committed lease, invalid finalized replica, or unbounded queue.
- Completion may decrease by no more than 0.5 percentage points.
- Median p50 and p95 may be at most 110% of matched baseline.
- Repo HA: 30/30 per accepted run, required W, zero invalid finalized replicas,
  and frozen repair coverage.
- UAV: no stale-session acceptance, unbounded pending bytes, or FEC regression.
- DI: zero conflicting committed role assignment and zero synthetic/untracked
  lease; authority loss ends within the child timeout budget.

Any relaxation requires a written tradeoff approved before seeing treatment
results. Performance cannot justify weakening correctness or security.
