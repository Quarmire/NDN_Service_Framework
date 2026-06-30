# Tasks: LLM MiniNDN Role Execution Adapter

## Phase 1: Planning Artifacts

- [x] T001 Create feature plan in `specs/030-llm-minindn-role-execution-adapter/plan.md`.
- [x] T002 Create task list in `specs/030-llm-minindn-role-execution-adapter/tasks.md`.

## Phase 2: Bundle Generator

- [x] T003 Add LLM proportional native bundle generator in `examples/python/NDNSF-DistributedInference/native_di_tracer/generate_llm_proportional_native_bundle.py`.
- [x] T004 Generate policy config, native execution plan, service manifest, controller policy, trust schema, and assignment summary from the 029 proportional plan.
- [x] T005 Ensure generated stage metadata uses deterministic runner scopes for linear stage dependencies and final response.

## Phase 3: Runtime Harness Adapter

- [x] T006 Add assignment CSV support to `examples/DI_NativePlanManifestSmoke.cpp`.
- [x] T007 Make `Experiments/NDNSF_DI_NativeTracer_Minindn.py` assignment CSV generation follow current plan roles.
- [x] T008 Add LLM proportional policy-bundle mode to `Experiments/NDNSF_DI_NativeTracer_Minindn.py`.
- [x] T009 Add deterministic runner option to provider check and serve commands.
- [x] T010 Validate full-network role timing against generated plan roles instead of fixed NativeTracer roles.

## Phase 4: Verification

- [x] T011 Compile changed Python files.
- [x] T012 Build affected C++ example targets.
- [x] T013 Generate an LLM proportional bundle and assignment CSV.
- [x] T014 Run schema smoke, manifest smoke, and provider check smoke for all LLM stage providers.
- [x] T015 Run `git diff --check` and CodeGraph sync/status.
- [x] T016 Run MiniNDN full-network LLM proportional execution.

## Evidence

- `PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile ...` passed for
  the LLM bundle generator, MiniNDN harness, and LLM planner.
- `./waf build --targets=di-native-plan-manifest-smoke,di-native-plan-schema-smoke,di-native-provider -j4`
  passed.
- `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/generate_llm_proportional_native_bundle.py --out /tmp/ndnsf-llm-native-bundle --summary-json /tmp/ndnsf-llm-native-bundle-summary.json`
  produced plan `6225694b14f15c4e` with roles `/LLM/Stage/0`,
  `/LLM/Stage/1`, and `/LLM/Stage/2`.
- `./build/examples/di-native-plan-schema-smoke /tmp/ndnsf-llm-native-bundle/native-execution-plan.json /Inference/NativeTracer llm onnx llm-pipeline`
  passed with 3 roles and 2 dependencies.
- `./build/examples/di-native-plan-manifest-smoke /tmp/ndnsf-llm-native-bundle/native-execution-plan.json /tmp/ndnsf-llm-native-bundle/service-manifest.json /Inference/NativeTracer --timing-csv /tmp/ndnsf-llm-native-bundle/local-execution-timing.csv --assignment llm-proportional --assignment-csv /tmp/ndnsf-llm-native-bundle/assignment.csv`
  passed with 3 roles, 3 artifacts, and 5 output tensors.
- Provider check passed for:
  `/NDNSF-DI/Tracer/provider/llm-2gb` running `/LLM/Stage/0`,
  `/NDNSF-DI/Tracer/provider/llm-4gb` running `/LLM/Stage/1`, and
  `/NDNSF-DI/Tracer/provider/llm-8gb` running `/LLM/Stage/2`.
- Harness local execution:
  `python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --policy-bundle llm-proportional --assignment llm-proportional --local-execution-only --out /tmp/ndnsf-llm-harness-local-2`
  returned `status=SUCCESS`, `localExecution=executed`, and
  `dependencyExecution=local-baseline-executed`.
- Final checks: `git diff --check` passed, and
  `codegraph sync . && codegraph status .` reported the index is up to date.
- Full-network MiniNDN execution:
  `sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --policy-bundle llm-proportional --assignment llm-proportional --full-network --requests 1 --concurrency 1 --out /tmp/ndnsf-llm-full-network`
  returned `status=SUCCESS`, `securityBootstrap=executed`,
  `userExecution=executed`, and `dependencyExecution=executed`.
