## Material Passport

- ID: `spec092-native-di-user-driver-correctness`
- Type: code experiment validation
- Verification Status: VERIFIED
- Runtime source: `dbb880c`
- Data access: local raw summaries and logs

# Validation Report

## Findings

| Finding | Status | Evidence |
|---|---|---|
| Threaded scope-key lifecycle defect is fixed | verified | 4 post-fix full-network runs completed 60/60 rather than failing scope-key fetch |
| Threaded driver sustains the 1 RPS screening point | verified | 3 matched runs, 180/180, mean 1.013324 RPS, worst slip 14.8 ms |
| Process-pool throughput denominator is corrected | verified | 59.273 s measurement interval excludes the five-second lead |
| Process-pool is scheduling-capable at 1 RPS | rejected | first-request slip 4345.5 ms exceeds the 1000 ms gate |
| Provider capacity is the first 1 RPS boundary | rejected | provider utilization stayed below 4%; threaded passes |
| Maximum stable RPS | not measured | one offered-load point only |

## Reproducibility

The three accepted threaded repetitions use the same commit and controls.
Result directories contain commands, policy bundles, assignments, logs,
machine-readable summaries, Qwen runtime-v1 evidence, and dependency counters.
All runs used a hard outer timeout and cleaned MiniNDN processes afterward.

## Fallacy Scan

Coverage: 11/11 checked.

- Simpson, ecological, Berkson, collider, base-rate, and reverse-causality
  patterns do not apply to this fixed matched systems treatment.
- Regression to the mean is reduced by three prespecified repetitions rather
  than selecting the best run.
- Survivorship is controlled by retaining the process-pool failure and both
  intermediate threaded measurement defects.
- Look-elsewhere and forking-path risk is bounded by the frozen 1 RPS gate;
  no alternate latency or adjusted throughput metric replaces a failed gate.
- Correlation is not promoted to general causation: the evidence supports this
  fixture and rate, not a maximum-RPS or universal driver claim.

## Verdict

The threaded user driver is the validated driver for the next Native DI
offered-load search. Process-pool needs explicit worker-ready synchronization
before it can be compared as a scheduling-capable driver. No provider/runtime
optimization is justified by this 1 RPS evidence.
