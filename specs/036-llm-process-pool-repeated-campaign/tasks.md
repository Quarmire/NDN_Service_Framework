# Tasks: LLM Process-Pool Repeated Campaign

## Phase 1: Setup

- [x] T001 Create feature plan and task list.
- [x] T002 Point Spec Kit active feature and agent context at feature 036.

## Phase 2: Campaign

- [x] T003 Run repeated process-pool MiniNDN campaign for 4 and 8 RPS.
- [x] T004 Parse the campaign summary into a markdown evidence table.

## Phase 3: Documentation And Verification

- [x] T005 Record interpretation in this task file.
- [x] T006 Run final checks: `git diff --check` and CodeGraph sync/status.

## Evidence

- Repeated campaign path:
  `/tmp/ndnsf-llm-process-pool-repeat-r4-r8`.
- Command:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_llm_full_network_campaign.py --out-root /tmp/ndnsf-llm-process-pool-repeat-r4-r8 --runs 5 --workloads base:40:8 --modes greedy,proportional --stage-execution-delay-scale 4 --target-rps-series 4,8 --open-loop-duration-s 5 --open-loop-driver-mode process-pool --provider-check-timeout 60`
- All 20 MiniNDN runs completed with `status=SUCCESS`. Every aggregate point
  has `submittedCount == scheduledRequestCount`, `successRate=1.0`, and
  `localBackpressureCount=0`.

  | mode | offered RPS | runs | scheduled | submitted | success | success rate | obs success RPS mean/std | p50 mean/std ms | p95 mean/std ms |
  | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
  | greedy | 4 | 5 | 100 | 100 | 100 | 1.00 | 1.436/0.011 | 1576.2/134.3 | 2073.1/51.0 |
  | greedy | 8 | 5 | 200 | 200 | 200 | 1.00 | 2.196/0.009 | 1656.2/30.6 | 2215.7/46.4 |
  | proportional | 4 | 5 | 100 | 100 | 100 | 1.00 | 1.487/0.022 | 1430.0/129.1 | 1948.1/125.6 |
  | proportional | 8 | 5 | 200 | 200 | 200 | 1.00 | 2.378/0.053 | 1372.4/109.6 | 1897.3/178.9 |

- Layouts:
  - `greedy` allocated all 28 layers to
    `/NDNSF-DI/Tracer/provider/llm-8gb`.
  - `proportional` allocated 4/8/16 layers to the 2GB/4GB/8GB providers.

## Interpretation

- Process-pool mode removes the local driver admission bottleneck for these
  runs: every scheduled request was submitted, so observed success RPS below
  offered RPS is now service-time/queueing behavior rather than local dropping.
- Under tiny Qwen NativeTracer and the current MiniNDN topology, proportional
  splitting is consistently better than greedy at 4 and 8 offered RPS:
  - 4 RPS: observed success RPS improves from 1.436 to 1.487, mean p50 drops
    from 1576.2 ms to 1430.0 ms, and mean p95 drops from 2073.1 ms to
    1948.1 ms.
  - 8 RPS: observed success RPS improves from 2.196 to 2.378, mean p50 drops
    from 1656.2 ms to 1372.4 ms, and mean p95 drops from 2215.7 ms to
    1897.3 ms.
- This is evidence for the current small-model harness, not yet a final claim
  about large LLMs. The next implementation step should expose provider-level
  utilization and queueing counters so the planner can explain why proportional
  helps under load.
