# Tasks: LLM Threaded Open-Loop Driver

## Phase 1: Planning

- [x] T001 Create feature plan in `specs/035-llm-threaded-open-loop-driver/plan.md`.
- [x] T002 Create task list in `specs/035-llm-threaded-open-loop-driver/tasks.md`.
- [x] T003 Point `.specify/feature.json` and agent context at feature 035.

## Phase 2: User Driver

- [x] T004 Add `--open-loop-driver-mode child|threaded|process-pool`.
- [x] T005 Implement fixed-rate threaded worker-pool scheduling.
- [x] T006 Report mode, submitted count, local backpressure, offered RPS, and per-request results.
- [x] T006A Implement fixed-rate process-pool scheduling after threaded MiniNDN failure.

## Phase 3: Harness And Campaign

- [x] T007 Pass open-loop driver mode through the MiniNDN harness.
- [x] T008 Pass open-loop driver mode through the LLM campaign runner and summaries.

## Phase 4: Verification

- [x] T009 Compile changed Python scripts and run CLI sanity.
- [x] T010 Run a short MiniNDN threaded smoke.
- [x] T011 Run a short MiniNDN process-pool smoke.
- [x] T012 Compare child vs process-pool behavior at 2 RPS and a sharper 8 RPS point.
- [x] T013 Record evidence and interpretation.
- [x] T014 Run `git diff --check` and CodeGraph sync/status.

## Evidence

- Threaded MiniNDN smoke:
  `/tmp/ndnsf-llm-threaded-smoke/greedy/base-r2/run-01/logs/user-driver.log`
  reached `NDNSF_DI_NATIVE_TRACER_USER_SUBMIT` for request 1, then exited
  without `NDNSF_DI_NATIVE_TRACER_USER_EXECUTION`. There was no Python
  traceback. This suggests multiple in-process native `ServiceUser` workers are
  not a reliable measurement path for MiniNDN campaigns.
- Process-pool MiniNDN smoke:
  `/tmp/ndnsf-llm-process-pool-smoke/greedy/base-r2/run-01` completed 4/4
  requests at offered 2 RPS with `submittedCount=4`, `localBackpressureCount=0`,
  and `successRate=1.0`.
- 2 RPS child comparison:
  `/tmp/ndnsf-llm-child-compare-r2/greedy/base-r2/run-01` also completed 4/4
  requests with no local backpressure, so 2 RPS is not a sharp enough point to
  show the old driver bottleneck.
- 8 RPS process-pool comparison:
  `/tmp/ndnsf-llm-process-pool-compare-r8/greedy/base-r8/run-01` completed
  16/16 scheduled requests with `submittedCount=16`, `localBackpressureCount=0`,
  and `successRate=1.0`.
- 8 RPS child comparison:
  `/tmp/ndnsf-llm-child-compare-r8/greedy/base-r8/run-01` submitted only 8/16
  scheduled requests and marked the other 8 as `local-open-loop-backpressure`,
  producing `successRate=0.5`. This confirms the old child driver was measuring
  local driver admission at high offered rates, while process-pool can drive the
  full scheduled workload into the NDNSF path.
- Process-pool 1/2/4/8 RPS sweep:
  `/tmp/ndnsf-llm-process-pool-rps-sweep-1-2-4-8` ran one 5-second MiniNDN
  campaign for `greedy` and `proportional`, with request cap 40 and concurrency
  8. Every point completed with `successRate=1.0`, `submittedCount` equal to
  `scheduledRequestCount`, and `localBackpressureCount=0`.

  | mode | offered RPS | scheduled | submitted | success | observed success RPS | p50 ms | p95 ms |
  | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
  | greedy | 1 | 5 | 5 | 5 | 0.475 | 1052.5 | 1319.9 |
  | greedy | 2 | 10 | 10 | 10 | 0.858 | 1820.9 | 1876.8 |
  | greedy | 4 | 20 | 20 | 20 | 1.442 | 1702.1 | 2057.2 |
  | greedy | 8 | 40 | 40 | 40 | 2.193 | 1701.3 | 2176.2 |
  | proportional | 1 | 5 | 5 | 5 | 0.480 | 1086.3 | 1226.7 |
  | proportional | 2 | 10 | 10 | 10 | 0.847 | 1849.4 | 1929.8 |
  | proportional | 4 | 20 | 20 | 20 | 1.505 | 1425.5 | 1830.7 |
  | proportional | 8 | 40 | 40 | 40 | 2.409 | 1308.7 | 1953.8 |

  The `greedy` layout used one 8GB provider for all 28 layers. The
  `proportional` layout used 2GB/4GB/8GB providers with 4/8/16 layers. This
  single-run sweep is not enough for final claims, but it confirms the driver
  no longer clips the workload locally and can expose layout-level behavior.

## Next Work

- Run a repeated process-pool campaign, for example 5-10 runs at 4 and 8 RPS,
  to report p50/p95/stddev across runs rather than one sample.
- Add provider utilization counters to the MiniNDN summary so proportional
  planning can be evaluated by both latency and provider work share.
- After repeated runs, decide whether proportional is genuinely better under
  high offered load or whether this one-run result is run-to-run noise.
