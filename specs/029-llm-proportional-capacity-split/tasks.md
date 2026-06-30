# Tasks: LLM Proportional Capacity Split

## Phase 1: Planning Artifacts

- [x] T001 Create feature plan in `specs/029-llm-proportional-capacity-split/plan.md`.
- [x] T002 Create task list in `specs/029-llm-proportional-capacity-split/tasks.md`.

## Phase 2: Profiles and Planner

- [x] T003 Add 2GB/4GB/8GB provider profile in `examples/python/NDNSF-DistributedInference/native_di_tracer/llm_provider_profiles_2_4_8.json`.
- [x] T004 Add matching ACK candidate sample in `examples/python/NDNSF-DistributedInference/native_di_tracer/llm_ack_candidates_2_4_8.json`.
- [x] T005 Add `--mode greedy|proportional` to `examples/python/NDNSF-DistributedInference/native_di_tracer/plan_llm_resource_aware.py`.
- [x] T006 Implement proportional layer allocation using normalized `min(memory ratio, compute ratio)`.
- [x] T007 Add plan summary fields for allocation ratios and bottleneck utilization.

## Phase 3: RPS Search

- [x] T008 Add greedy/proportional RPS search script in `examples/python/NDNSF-DistributedInference/native_di_tracer/run_llm_proportional_rps_search.py`.
- [x] T009 Run planner-derived RPS search and record outputs.
- [x] T010 Attempt MiniNDN compatibility search or document why current harness cannot execute LLM roles.

## Phase 4: Verification

- [x] T011 Validate greedy 2/4/8 imbalance.
- [x] T012 Validate proportional 2/4/8 approximate 1:2:4 split.
- [x] T013 Validate ACK-derived proportional planning.
- [x] T014 Run Python compile, `git diff --check`, and CodeGraph sync/status.

## Evidence

- Greedy 2/4/8 validation:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_llm_resource_aware.py --model-spec examples/python/NDNSF-DistributedInference/native_di_tracer/llm_model_spec_qwen_tiny_proportional.json --provider-profiles examples/python/NDNSF-DistributedInference/native_di_tracer/llm_provider_profiles_2_4_8.json --out /tmp/llm-greedy-248.json --mode greedy --validate --expect-shards no`
  produced allocation `{"/NDNSF-DI/Tracer/provider/llm-8gb": 28}`.
- Proportional 2/4/8 validation:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_llm_resource_aware.py --model-spec examples/python/NDNSF-DistributedInference/native_di_tracer/llm_model_spec_qwen_tiny_proportional.json --provider-profiles examples/python/NDNSF-DistributedInference/native_di_tracer/llm_provider_profiles_2_4_8.json --out /tmp/llm-proportional-248.json --mode proportional --validate --expect-shards no`
  produced allocation `{"/NDNSF-DI/Tracer/provider/llm-2gb": 4, "/NDNSF-DI/Tracer/provider/llm-4gb": 8, "/NDNSF-DI/Tracer/provider/llm-8gb": 16}`.
- ACK-derived proportional validation:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_llm_resource_aware.py --model-spec examples/python/NDNSF-DistributedInference/native_di_tracer/llm_model_spec_qwen_tiny_proportional.json --ack-candidates-json examples/python/NDNSF-DistributedInference/native_di_tracer/llm_ack_candidates_2_4_8.json --out /tmp/llm-proportional-ack-248.json --mode proportional --validate --expect-shards no`
  produced the same `4/8/16` allocation.
- RPS search:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_llm_proportional_rps_search.py --out-root /tmp/ndnsf-llm-proportional-rps --target-rps-list 1,5,10,20,30,40`
  wrote `/tmp/ndnsf-llm-proportional-rps/llm-proportional-rps-summary.json`
  and `/tmp/ndnsf-llm-proportional-rps/llm-proportional-rps-search.csv`.
  Planner-derived max stable RPS: greedy `16.19`, proportional `28.333`.
- MiniNDN note:
  The current MiniNDN full-network harness executes `/Inference/NativeTracer`;
  it does not yet execute LLM proportional roles, so this feature records
  planner-derived RPS evidence and documents the runtime gap rather than
  claiming a false LLM MiniNDN execution.
- Final checks:
  `PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile ...` passed.
  `git diff --check` passed.
  `codegraph sync . && codegraph status .` passed.
