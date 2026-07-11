## Material Passport

- ID: `spec093-native-di-threaded-rps-boundary`
- Type: code experiment validation
- Verification Status: VERIFIED for 1-8 RPS tested range
- Runtime source: threaded driver from `dbb880c`; final harness `b9f47ab`
- Data access: local raw summaries and logs

# Validation Report

## Findings

| Finding | Status | Evidence |
|---|---|---|
| Threaded driver remains stable at 2 RPS | verified | 120/120, 2.009 RPS, 14.9 ms slip, 480 dependencies |
| Threaded driver remains stable at 4 RPS | verified | 240/240, 4.002 RPS, 15.7 ms slip, 960 dependencies |
| Threaded driver remains stable at 8 RPS | verified in three matched runtime treatments | 1440/1440, mean 7.985 RPS, worst 16.24 ms slip |
| Maximum stable RPS is 8 | not supported | no unstable point was measured above 8 RPS |
| Provider capacity is the current boundary | rejected within tested range | busiest stage about 26%, queue/pending at most one |
| Concurrent plain-text trace writes can interleave | verified | bounded ACK/dependency parse errors at 8 RPS |

## Reproducibility

Commands, controls, request caps, results, summaries, and anomaly diagnosis are
recorded under Spec 093. Each treatment used a hard timeout and MiniNDN cleanup.
The 8 RPS treatments were replayed through the final strict collectors so raw
service evidence and observability corruption remain distinguishable.

## Fallacy Scan

Coverage: 11/11 checked.

- Simpson, ecological, Berkson, collider, base-rate, and reverse-causality
  patterns are not applicable to the fixed single-fixture systems treatments.
- Regression-to-mean and survivorship risks are controlled by three 8 RPS
  repetitions and retention of both post-processing anomalies.
- Look-elsewhere and forking-path risk is bounded by stopping at the
  predeclared 8 RPS ceiling rather than extending to 16 RPS after success.
- Correlation is not promoted to universal causation: the result is explicitly
  limited to this Qwen model, layout, topology, concurrency, and tested range.

## Verdict

The validated threaded driver supports at least 8 offered RPS on this fixture.
No limiting layer was reached. A future feature may deliberately extend the
range, but 8 RPS must be reported only as the highest tested stable point.
