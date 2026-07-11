# MiniNDN Results

All final classifications use the Spec 091 gates: submit at least 95%, maximum
slip below one 1,000 ms request interval, no local backpressure, success at
least 99%, achieved throughput at least 95% of offered load, and complete
dependency execution.

## Screening

| Driver | Requests | Throughput | p50 / p95 | Max slip | Dependencies | Classification |
|---|---:|---:|---:|---:|---:|---|
| threaded-fixed | 60/60 | 1.013 RPS | 222.1 / 1175.6 ms | 12.2 ms | 240/240 | passes |
| process-pool | 60/60 | 1.012 RPS | 213.5 / 1182.2 ms | 4345.5 ms | 240/240 | fails scheduling gate |

The process-pool run is a useful negative result. The first five requests had
slips of 4345.5, 3341.1, 2387.1, 1418.4, and 817.3 ms. Later requests mostly
returned to below 17 ms, so aggregate throughput hid a fixed worker bootstrap
delay. A fixed five-second lead is not a readiness protocol.

The first corrected threaded run at commit `387f493` established 60/60 success
after keeping the base scope-key producer running, but exposed teardown in the
throughput denominator. The next run at `8d0b62a` established the corrected
driver measurement. Final matched repetitions use `dbb880c`, which also
preserves measurement metadata in the top-level harness summary.

## Threaded Repetitions

| Run | Success | Throughput | p50 | p95 | Max slip | Measurement elapsed |
|---|---:|---:|---:|---:|---:|---:|
| rep1 | 60/60 | 1.013351 RPS | 218.3 ms | 1177.9 ms | 14.8 ms | 59.209 s |
| rep2 | 60/60 | 1.013296 RPS | 208.0 ms | 1177.6 ms | 13.1 ms | 59.213 s |
| rep3 | 60/60 | 1.013326 RPS | 209.1 ms | 1173.7 ms | 14.1 ms | 59.211 s |

Across the three runs: 180/180 requests and 720/720 dependency events
completed. Mean throughput was 1.013324 RPS (population standard deviation
0.000023), mean p50 was 211.8 ms (SD 4.6), mean p95 was 1176.4 ms (SD 1.9),
and the worst maximum slip was 14.8 ms. No local backpressure or negative ACK
occurred.

Raw summaries:

```text
results/spec092-native-di-user-driver-correctness/process-pool/summary.json
results/spec092-native-di-user-driver-correctness/threaded-rep1/summary.json
results/spec092-native-di-user-driver-correctness/threaded-rep2/summary.json
results/spec092-native-di-user-driver-correctness/threaded-rep3/summary.json
```

The MiniNDN environment prints that its dummy keychain patch disables MiniNDN
security enforcement. Application bootstrap, typed capability envelopes, and
the NDNSF collaboration/dependency path executed, but these performance runs
are not standalone cryptographic-strength validation.
