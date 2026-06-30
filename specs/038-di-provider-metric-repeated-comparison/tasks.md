# Tasks: DI Provider Metric Repeated Comparison

## Phase 1: Setup

- [x] T001 Create feature 038 plan and task list.
- [x] T002 Point Spec Kit active feature and agent context at feature 038.

## Phase 2: Campaign

- [x] T003 Run repeated provider-metric MiniNDN campaign for 4 and 8 RPS.
- [x] T004 Parse aggregate provider utilization and latency results.

## Phase 3: Interpretation And Verification

- [x] T005 Record comparison evidence and interpretation.
- [x] T006 Run final checks: `git diff --check` and CodeGraph sync/status.

## Evidence

- Repeated campaign path:
  `/tmp/ndnsf-llm-provider-metric-repeat-r4-r8`.
- Command:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_llm_full_network_campaign.py --out-root /tmp/ndnsf-llm-provider-metric-repeat-r4-r8 --runs 5 --workloads base:40:8 --modes greedy,proportional --stage-execution-delay-scale 4 --target-rps-series 4,8 --open-loop-duration-s 5 --open-loop-driver-mode process-pool --provider-check-timeout 60`
- All 20 MiniNDN runs completed successfully. Every aggregate point has
  `successRate=1.0`, `submittedCount == scheduledRequestCount`, and
  `localBackpressureCount=0`.

## Latency And Throughput

| mode/rate | success | obs RPS | p50 ms | p95 ms | local BP |
| --- | ---: | ---: | ---: | ---: | ---: |
| greedy/base-r4 | 1.00 | 1.439 | 1646.9 | 2078.4 | 0 |
| proportional/base-r4 | 1.00 | 1.498 | 1323.2 | 1869.1 | 0 |
| greedy/base-r8 | 1.00 | 2.193 | 1654.8 | 2337.7 | 0 |
| proportional/base-r8 | 1.00 | 2.447 | 1296.1 | 1910.6 | 0 |

## Provider Metrics

| mode/rate | provider | roles | events | util | queue mean ms | queue max ms | handler mean ms | pending max |
| --- | --- | --- | ---: | ---: | ---: | ---: | ---: | ---: |
| greedy/base-r4 | llm-8gb | 0,1 | 200 | 0.996 | 102.5 | 128.1 | 108.2 | 2 |
| proportional/base-r4 | llm-2gb | 0 | 100 | 0.747 | 93.2 | 126.2 | 123.6 | 1 |
| proportional/base-r4 | llm-4gb | 1 | 100 | 0.738 | 15.4 | 94.5 | 123.1 | 1 |
| proportional/base-r4 | llm-8gb | 2 | 100 | 0.733 | 16.4 | 96.4 | 122.3 | 1 |
| greedy/base-r8 | llm-8gb | 0,1 | 400 | 0.998 | 104.1 | 129.7 | 108.1 | 2 |
| proportional/base-r8 | llm-2gb | 0 | 200 | 0.755 | 91.9 | 121.5 | 121.2 | 2 |
| proportional/base-r8 | llm-4gb | 1 | 200 | 0.756 | 25.4 | 115.4 | 121.2 | 2 |
| proportional/base-r8 | llm-8gb | 2 | 200 | 0.750 | 14.7 | 114.7 | 120.5 | 1 |

## Interpretation

- The driver is not clipping the workload: all scheduled requests are submitted
  and there is no local backpressure.
- Greedy overloads a single 8GB provider. Its estimated utilization is almost
  fully saturated (`0.996` at 4 RPS and `0.998` at 8 RPS), and queue wait stays
  around 102-104 ms.
- Proportional splits work across the 2GB/4GB/8GB providers. Utilization is
  spread around 0.73-0.76, and the later-stage providers have much lower mean
  queue wait than greedy.
- Proportional has a slightly higher per-role handler mean because each stage
  is represented as a separate provider-side role, but it still improves p50,
  p95, and observed success RPS under offered load because it reduces the
  single-provider queue.
- This supports the planner direction: for concurrent LLM service requests,
  capacity-aware proportional stage assignment can increase effective
  throughput even for tiny Qwen, while greedy single-provider execution becomes
  queue-limited.
