# Tasks: DI Planner Prediction Alignment

## Phase 1: Setup

- [x] T001 Create feature 039 plan and task list.
- [x] T002 Point Spec Kit active feature and agent context at feature 039.

## Phase 2: Planner Evidence

- [x] T003 Add provider load, queue risk, and dependency cost prediction to `examples/python/NDNSF-DistributedInference/native_di_tracer/plan_llm_resource_aware.py`.
- [x] T004 Pass `targetRps` and provider worker assumptions through `examples/python/NDNSF-DistributedInference/native_di_tracer/generate_llm_proportional_native_bundle.py` and `Experiments/NDNSF_DI_NativeTracer_Minindn.py`.

## Phase 3: Campaign Reporting

- [x] T005 Include planner prediction evidence in `summary.json`, campaign CSV rows, and aggregate campaign summaries.

## Phase 4: Verification

- [x] T006 Generate greedy/proportional planner JSON at 8 RPS and inspect prediction fields.
- [x] T007 Run a short MiniNDN process-pool campaign to verify prediction evidence survives end to end.
- [x] T008 Run final checks: Python compile, `git diff --check`, and CodeGraph sync/status.

## Evidence

- Planner JSON smoke commands:
  - `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_llm_resource_aware.py --model-spec examples/python/NDNSF-DistributedInference/native_di_tracer/llm_model_spec_qwen_tiny_proportional.json --provider-profiles examples/python/NDNSF-DistributedInference/native_di_tracer/llm_provider_profiles_2_4_8.json --mode greedy --target-rps 8 --prediction-compute-scale 4 --out /tmp/ndnsf-greedy-plan-prediction.json --validate --expect-shards no`
  - `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_llm_resource_aware.py --model-spec examples/python/NDNSF-DistributedInference/native_di_tracer/llm_model_spec_qwen_tiny_proportional.json --provider-profiles examples/python/NDNSF-DistributedInference/native_di_tracer/llm_provider_profiles_2_4_8.json --mode proportional --target-rps 8 --prediction-compute-scale 4 --out /tmp/ndnsf-proportional-plan-prediction.json --validate --expect-shards no`
- Planner JSON smoke result at 8 offered RPS:
  - greedy: predicted bottleneck `/NDNSF-DI/Tracer/provider/llm-8gb`, max predicted utilization `1.68`, queue risk `saturated`.
  - proportional: predicted load spread across `llm-2gb`, `llm-4gb`, and `llm-8gb`, max predicted utilization `0.96`, queue risk `high`.
  - Dependency transfer is reported separately from provider service time, so provider utilization prediction tracks execution pressure rather than network transfer cost.
- MiniNDN process-pool smoke path:
  `/tmp/ndnsf-llm-planner-prediction-smoke`.
- MiniNDN command:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_llm_full_network_campaign.py --out-root /tmp/ndnsf-llm-planner-prediction-smoke --runs 1 --workloads base:8:4 --modes greedy,proportional --stage-execution-delay-scale 4 --target-rps 4 --open-loop-duration-s 2 --open-loop-driver-mode process-pool --provider-check-timeout 60`
- MiniNDN smoke result:
  - greedy/base-r4: success `8/8`, local backpressure `0`, planner bottleneck `llm-8gb`, predicted utilization `0.84`, measured provider utilization `1.0`.
  - proportional/base-r4: success `8/8`, local backpressure `0`, planner predicted utilization `0.48` per active provider, measured provider utilization around `0.72`.
  - `plannerPrediction` is present in aggregate
    `/tmp/ndnsf-llm-planner-prediction-smoke/llm-full-network-campaign-summary.json`.
- Final checks:
  - `PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile ...` passed for touched Python files.
  - `git diff --check` passed.
  - `.specify/extensions/agent-context/scripts/bash/update-agent-context.sh specs/039-di-planner-prediction-alignment/plan.md` completed.
  - `codegraph sync . && codegraph status .` completed with the index up to date.
