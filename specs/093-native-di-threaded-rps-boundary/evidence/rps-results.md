# Threaded Offered-Load Results

## Coarse Search

| Target | Requests | Success | Throughput | p50 / p95 | Max slip / interval | Dependency evidence | Verdict |
|---:|---:|---:|---:|---:|---:|---:|---|
| 1 RPS | 180 | 180/180 | 1.0133 mean | 211.8 / 1176.4 ms mean | 14.8 / 1000 ms worst | 720/720 | stable anchor |
| 2 RPS | 120 | 120/120 | 2.0090 | 216.3 / 680.9 ms | 14.9 / 500 ms | 480/480 | stable |
| 4 RPS | 240 | 240/240 | 4.0022 | 216.5 / 434.2 ms | 15.7 / 250 ms | 960/960 | stable |
| 8 RPS | 1440 | 1440/1440 | 7.9850 mean | 198.5 / 247.6 ms mean | 16.2 / 125 ms worst | 5760 markers | stable through tested ceiling |

All points submitted 100% of scheduled requests, had zero local backpressure,
zero request failure, and achieved more than 95% of target throughput. No first
unstable point appeared within the predeclared 1-8 RPS range, so there is no
bracket to bisect. The search stops at its planned coarse ceiling rather than
expanding the hypothesis after seeing favorable results.

## 8 RPS Repetitions

| Run | Success | Throughput | p50 | p95 | Max slip | Dependency trace | ACK hint trace |
|---|---:|---:|---:|---:|---:|---|---|
| run1 | 480/480 | 7.9887 | 198.1 ms | 249.2 ms | 15.92 ms | 1920 valid | 2867 valid + 1 parse error |
| run2 | 480/480 | 7.9795 | 198.0 ms | 251.5 ms | 16.24 ms | 1919 valid + 1 parse error | 2867 valid + 1 parse error |
| run3 | 480/480 | 7.9866 | 199.3 ms | 241.9 ms | 15.03 ms | 1920 valid | 2867 valid + 1 parse error |

Population statistics across the three runs:

```text
throughput mean 7.984960 RPS, SD 0.003935
p50 mean 198.468 ms, SD 0.578
p95 mean 247.552 ms, SD 4.110
maximum-slip mean 15.730 ms, SD 0.516; worst 16.244 ms
```

The three treatments have identical runtime controls. Harness-only
post-processing fixes were introduced between runs to preserve malformed log
records; they execute after the MiniNDN network workload and do not change the
user/provider runtime path. Run1's failed top-level post-processing and run2's
damaged dependency line remain in the evidence rather than being discarded.

At run3, the busiest providers were the 4 GB and 8 GB stages at approximately
26% estimated utilization. Queue/pending peaks were at most one. This is the
highest observed pressure, not a limiting layer: no scheduling, completion,
throughput, dependency, or provider-capacity gate failed.

Raw paths:

```text
results/spec093-native-di-threaded-rps-boundary/rps-2-run1
results/spec093-native-di-threaded-rps-boundary/rps-4-run1
results/spec093-native-di-threaded-rps-boundary/rps-8-run1
results/spec093-native-di-threaded-rps-boundary/rps-8-run2
results/spec093-native-di-threaded-rps-boundary/rps-8-run3
```

The MiniNDN dummy-keychain warning remains. These runs validate application
bootstrap and the real Qwen NDNSF collaboration/dependency path, but are not a
standalone cryptographic-strength evaluation.
