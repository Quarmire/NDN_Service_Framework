# Experiment And Performance Gates

Thresholds are frozen in the baseline evidence before treatment code is run.
They may be tightened later, but a relaxation requires a written tradeoff
decision made independently of the treatment result.

## Default Matched Campaign

- Same commit inputs, topology, provider profiles, load, duration, warmup,
  random-seed set, logging level, and environment.
- At least three independent 60-second measured runs per condition after
  warmup. Research-retention decisions such as advisory coordination use at
  least ten matched runs.
- Record raw per-run completion, failures by reason, p50, p95, throughput,
  resource use, and exact commands/environment.
- Report median and range; research-retention decisions additionally report a
  95% bootstrap confidence interval for the paired effect.

## Blocking Defaults

- All correctness and security tests pass; no authorization bypass, conflicting
  committed lease, invalid finalized replica, or unbounded queue is permitted.
- Completion rate may not decrease by more than 0.5 percentage points from a
  canonical baseline unless the child spec pre-approves a stricter correctness
  tradeoff.
- Median p50 and p95 may not exceed 110% of baseline. A child spec may define a
  tighter bound for a known hot path.
- Repo HA retains 30/30 completion per run, required W, zero invalid finalized
  replicas, and the frozen repair-coverage target.
- Performance alone cannot justify removing a correctness or security
  mechanism. An unexplained regression blocks deletion.

The baseline record must name the canonical result directory and explain any
deviation from these defaults before implementation begins.
