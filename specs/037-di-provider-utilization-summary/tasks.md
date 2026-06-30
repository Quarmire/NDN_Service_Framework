# Tasks: DI Provider Utilization Summary

## Phase 1: Setup

- [x] T001 Create feature 037 plan and task list.
- [x] T002 Point Spec Kit active feature and agent context at feature 037.

## Phase 2: Implementation

- [x] T003 Parse provider timing and capacity logs in the MiniNDN harness.
- [x] T004 Add provider utilization to each run's `summary.json`.
- [x] T005 Carry provider utilization through campaign rows and aggregate summary.

## Phase 3: Verification

- [x] T006 Compile changed Python scripts.
- [x] T007 Run a small MiniNDN process-pool smoke and inspect provider metrics.
- [x] T008 Record evidence and interpretation.
- [x] T009 Run final checks: `git diff --check` and CodeGraph sync/status.

## Evidence

- Python compile:
  `PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile Experiments/NDNSF_DI_NativeTracer_Minindn.py examples/python/NDNSF-DistributedInference/native_di_tracer/run_llm_full_network_campaign.py`
- MiniNDN smoke:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_llm_full_network_campaign.py --out-root /tmp/ndnsf-llm-provider-util-smoke --runs 1 --workloads base:8:4 --modes proportional --stage-execution-delay-scale 4 --target-rps 4 --open-loop-duration-s 2 --open-loop-driver-mode process-pool --provider-check-timeout 60`
- Smoke result path:
  `/tmp/ndnsf-llm-provider-util-smoke`.
- The smoke completed with `successRate=1.0`, `submittedCount=8`, and
  `localBackpressureCount=0`.

  | provider | roles | events | sessions | util | queue mean/max ms | handler mean ms | pending max | waiting inputs max |
  | --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
  | `/NDNSF-DI/Tracer/provider/llm-2gb` | `/LLM/Stage/0` | 8 | 8 | 0.747 | 78.238/131.164 | 126.807 | 1 | 0 |
  | `/NDNSF-DI/Tracer/provider/llm-4gb` | `/LLM/Stage/1` | 8 | 8 | 0.731 | 0.466/1.570 | 123.292 | 1 | 1 |
  | `/NDNSF-DI/Tracer/provider/llm-8gb` | `/LLM/Stage/2` | 8 | 8 | 0.725 | 0.040/0.094 | 123.151 | 1 | 1 |

## Interpretation

- The new parser confirms that proportional mode actually exercised all three
  configured providers: 2GB, 4GB, and 8GB each handled 8 role events.
- The smoke exposes provider-level bottleneck shape. In this short run,
  `/LLM/Stage/0` on the 2GB provider had the highest queue wait, while later
  stages mostly waited on dependency availability rather than local queueing.
- Campaign summaries now contain enough data to explain future greedy vs
  proportional results without manually grepping provider logs.
