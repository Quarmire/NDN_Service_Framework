# Frozen Experiment Gate

Primary metric: provider admission lease-conflict rate.

- At least ten matched seeds, topology, request sequence, offered load, and
  timeout settings for coordinator-off and advisory variants.
- Retain advisory only when paired improvement is >=10% and the paired 95%
  bootstrap confidence interval excludes zero.
- Completion must not fall below the coordinator-off baseline threshold and
  p95 must not exceed its frozen threshold.
- Also report completion, p50, p95, stable RPS, and advisory hop cost.
- Negative or inconclusive evidence causes deletion, not reinterpretation.
