# Tasks: LLM Resource-Aware Planning

## Phase 1: Planning Artifacts

- [x] T001 Create feature design document in `specs/027-llm-resource-aware-planning/plan.md`.
- [x] T002 Create executable task list in `specs/027-llm-resource-aware-planning/tasks.md`.

## Phase 2: Reusable Planner

- [x] T003 [P] Add sample Qwen-small model spec in `examples/python/NDNSF-DistributedInference/native_di_tracer/llm_model_spec_qwen_small.json`.
- [x] T004 [P] Add sample MiniNDN provider resource profiles in `examples/python/NDNSF-DistributedInference/native_di_tracer/llm_provider_profiles.json`.
- [x] T005 Implement reusable LLM resource-aware planner in `examples/python/NDNSF-DistributedInference/native_di_tracer/plan_llm_resource_aware.py`.
- [x] T006 Validate normal linear-pipeline output and forced-sharding output using the planner CLI.

## Phase 3: MiniNDN ACK Resource Metadata

- [x] T007 Append preconfigured provider resource fields to native provider ACK payloads in `NDNSF-DistributedInference/cpp/ndnsf-di/NativeProviderReadiness.cpp`.
- [x] T008 Add per-provider resource environment overrides to `Experiments/NDNSF_DI_NativeTracer_Minindn.py`.
- [x] T009 Record provider resource profiles in MiniNDN summary output.

## Phase 4: Verification

- [x] T010 Compile Python planner and harness files.
- [x] T011 Build the native DI provider target.
- [x] T012 Run `git diff --check` and record final evidence here.

## Phase 5: Review Closure

- [x] T013 Document implementation review findings in `specs/027-llm-resource-aware-planning/plan.md`.
- [x] T014 Add ACK-candidate sample input in `examples/python/NDNSF-DistributedInference/native_di_tracer/llm_ack_candidates_sample.json`.
- [x] T015 Teach `examples/python/NDNSF-DistributedInference/native_di_tracer/plan_llm_resource_aware.py` to derive provider profiles from ACK payloads.
- [x] T016 Add reusable plan cache support to `examples/python/NDNSF-DistributedInference/native_di_tracer/plan_llm_resource_aware.py`.
- [x] T017 Fix comma-separated `modelFamilies` parsing in `examples/python/NDNSF-DistributedInference/native_di_tracer/plan_llm_resource_aware.py`.
- [x] T018 Validate ACK-derived planning and cache reuse.
- [x] T019 Re-run Python compile, native provider build, `git diff --check`, and CodeGraph sync.

## Evidence

- Planner normal path:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_llm_resource_aware.py --model-spec examples/python/NDNSF-DistributedInference/native_di_tracer/llm_model_spec_qwen_small.json --provider-profiles examples/python/NDNSF-DistributedInference/native_di_tracer/llm_provider_profiles.json --out /tmp/ndnsf_llm_plan.json --validate --expect-shards no`
  produced `stages=2`, `shards=0`.
- Planner forced-capacity path:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_llm_resource_aware.py --model-spec /tmp/ndnsf_llm_forced_model.json --provider-profiles examples/python/NDNSF-DistributedInference/native_di_tracer/llm_provider_profiles.json --out /tmp/ndnsf_llm_forced_plan.json --validate --expect-shards yes`
  produced `stages=0`, `shards=24`.
- Python compile:
  `PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile examples/python/NDNSF-DistributedInference/native_di_tracer/plan_llm_resource_aware.py Experiments/NDNSF_DI_NativeTracer_Minindn.py NDNSF-DistributedInference/ndnsf_distributed_inference/planner_registry.py`
- Native provider build:
  `./waf build --targets=di-native-provider -j4`
- Workspace checks:
  `git diff --check`
  `codegraph sync .`
  `codegraph status .`
- Review closure:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_llm_resource_aware.py --model-spec examples/python/NDNSF-DistributedInference/native_di_tracer/llm_model_spec_qwen_small.json --provider-profiles examples/python/NDNSF-DistributedInference/native_di_tracer/llm_provider_profiles.json --out /tmp/ndnsf_llm_plan_profile.json --cache-dir /tmp/ndnsf_llm_plan_cache --validate --expect-shards no`
  produced `planId=c3841c02b1b2c2b6`, `stages=2`, `shards=0`, `cacheHit=false`.
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_llm_resource_aware.py --model-spec examples/python/NDNSF-DistributedInference/native_di_tracer/llm_model_spec_qwen_small.json --ack-candidates-json examples/python/NDNSF-DistributedInference/native_di_tracer/llm_ack_candidates_sample.json --out /tmp/ndnsf_llm_plan_ack_1.json --cache-dir /tmp/ndnsf_llm_plan_cache --validate --expect-shards no`
  produced `planId=6a341149a3bad6b1`, `stages=2`, `shards=0`, `cacheHit=false`.
  A second ACK-derived run with the same inputs produced the same
  `planId=6a341149a3bad6b1` and `cacheHit=true`.
  Forced-capacity validation produced `stages=0`, `shards=24`.
  Final checks passed:
  `PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile ...`,
  `./waf build --targets=di-native-provider -j4`, `git diff --check`,
  `codegraph sync .`, and `codegraph status .`.
