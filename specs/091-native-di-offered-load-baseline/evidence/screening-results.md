# Native DI Offered-Load Screening Results

All runs used commit `30109fe`, Qwen ONNX NativeTracer, AI_Lab topology,
proportional 2/4/8 GB placement, 1 offered RPS, 60 seconds, 60 requests, and
concurrency 4. Only `open_loop_driver_mode` and its output path differed.

| Mode | Root status | Results | Success | Throughput | p50 / p95 | Max slip | Local waits | Dependency events |
|---|---|---:|---:|---:|---:|---:|---:|---:|
| child | SUCCESS | 60 | 60/60 | 0.410 RPS | 275.8 / 317.3 ms | 77,961 ms | 1,288 | 240 |
| threaded | FAILURE | 23 partial log rows | 0 completed summary | n/a | n/a | >69,000 ms observed | blocked | 0 |
| process-pool | SUCCESS | 60 | 60/60 | 0.932 RPS | 215.8 / 1,175.8 ms | not instrumented | 0 | 240 |

## Gate Classification

- `child`: not scheduling-capable. It submits all requests eventually, but
  per-request process setup/backpressure stretches makespan to 146.2 seconds.
- `threaded`: not scheduling-capable and not a valid full-network result. Four
  initial requests block; later rows report that worker users cannot fetch the
  base user's scope-key large Data. The harness correctly emits FAILURE because
  no final execution JSON exists.
- `process-pool`: best candidate and all network work completes. Provider
  pending work peaks at one, ready queue at zero, and maximum observed provider
  utilization is 3.87%, so provider capacity is not the first boundary at
  1 RPS. It does not pass the predeclared stable gate because reported
  throughput is 93.2%, below 95%.

The process-pool makespan includes its intentional five-second future schedule
start. Removing that fixed lead for diagnosis gives 60 / (64.368 - 5) = 1.011
RPS, but this adjusted number is not accepted evidence until the harness records
the measurement interval directly. Process-pool also reports max schedule slip
as zero without collecting worker slip, so the scheduling gate is incomplete.

## First Limiting Layer

The first measured boundary is the user-driver layer, not provider admission,
dependency exchange, or model execution. Both successful modes complete all
240 expected dependency events. Provider queues remain empty or at one while
the child driver loses most offered throughput to local scheduling.

Raw summaries:

```text
results/spec091-native-di-offered-load-baseline/child/summary.json
results/spec091-native-di-offered-load-baseline/threaded/summary.json
results/spec091-native-di-offered-load-baseline/process-pool/summary.json
```

