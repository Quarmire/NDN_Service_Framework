## Material Passport

- ID: `spec091-native-di-offered-load-screening`
- Type: code experiment validation
- Status: VERIFIED for reproducibility of the three commands; screening only
- Runtime source: `30109fe`
- Data access: local raw JSON and logs

# Validation Report

## Reproducibility

The commands resolved to identical topology, Qwen artifacts, assignment,
target, duration, concurrency, request cap, security path, and telemetry. The
three machine-readable summaries and raw logs exist. No stale MiniNDN process
was present before a treatment, and each harness cleaned its processes before
the next treatment.

## Findings

| Finding | Confidence | Evidence |
|---|---|---|
| Child driver is the first throughput boundary | high for this screening point | 77.96 s slip and 0.410 RPS with low provider utilization |
| Threaded mode has a lifecycle/data-serving defect | high | no execution summary; worker large-data scope-key fetch failures |
| Process-pool is the best current candidate | moderate | 60/60, zero backpressure, 240 dependency events |
| Process-pool meets 1 RPS | unresolved | reported 0.932 RPS includes 5 s lead and lacks worker slip telemetry |
| Maximum stable RPS | not measured | only one screening rate and one run per mode |

## Fallacy Scan

Coverage: 11/11 checked.

- Simpson/ecological/Berkson/collider/base-rate/reverse-causality: not
  applicable to this fixed matched systems screening.
- Regression to mean: no extreme-case selection claim.
- Survivorship: threaded failure and partial rows are retained, not dropped.
- Look-elsewhere/forking paths: thresholds and the three modes were frozen
  before execution; no best-looking metric is substituted.
- Correlation/causation: the driver treatment is controlled, but one run per
  mode supports only boundary screening, not a general performance claim.

## Decision

Open Spec 092 with two test-first fixes:

1. keep the base scope-key producer running during threaded workers;
2. record process-pool measurement start, per-worker schedule slip, and
   throughput excluding the intentional schedule lead.

Then rerun threaded and process-pool at the same point. If process-pool passes,
repeat it at least three times before any rate search or paper-facing claim.

