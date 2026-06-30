# Tasks: LLM Capacity Pressure Campaign

## Phase 1: Planning Artifacts

- [x] T001 Create feature plan in `specs/032-llm-capacity-pressure-campaign/plan.md`.
- [x] T002 Create task list in `specs/032-llm-capacity-pressure-campaign/tasks.md`.

## Phase 2: Delay Plumbing

- [x] T003 Make `examples/DI_NativeProviderExecutable.cpp` deterministic runner honor `executionDelayMs`.
- [x] T004 Add `--stage-execution-delay-ms` and `--stage-execution-delay-scale` to `generate_llm_proportional_native_bundle.py`.
- [x] T005 Pass harness `--role-execution-delay-ms` and `--llm-stage-execution-delay-scale` into the LLM bundle generator.
- [x] T006 Add delay controls to `run_llm_full_network_campaign.py`.

## Phase 3: Verification

- [x] T007 Compile changed Python scripts.
- [x] T008 Rebuild `di-native-provider`.
- [x] T009 Run a delayed greedy/proportional full-network campaign.
- [x] T010 Record campaign evidence and interpretation.
- [x] T011 Run `git diff --check` and CodeGraph sync/status.

## Evidence

- Python compile passed:
  `PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile ...`.
- C++ rebuild passed:
  `./waf build --targets=di-native-provider,di-native-plan-manifest-smoke -j4`.
- Local delay validation:
  `python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --policy-bundle llm-proportional --assignment llm-proportional --llm-planner-mode proportional --role-execution-delay-ms 120 --local-execution-only --out /tmp/ndnsf-llm-delay-local`
  produced local stage execute times near 120 ms.
- Fixed-delay pressure campaign:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_llm_full_network_campaign.py --out-root /tmp/ndnsf-llm-pressure-campaign --runs 1 --workloads c4:4:4 --modes greedy,proportional --role-execution-delay-ms 120`
  completed both modes with 4/4 success. Greedy p50/p95 were
  `468.427/473.232 ms`; proportional p50/p95 were `559.208/613.928 ms`.
  This fixed-delay setup is intentionally recorded as unfair to proportional,
  because it gives each stage the same delay even though proportional has more
  stages per request.
- Scale-based pressure campaign:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_llm_full_network_campaign.py --out-root /tmp/ndnsf-llm-pressure-scale-campaign --runs 1 --workloads c4:4:4 --modes greedy,proportional --stage-execution-delay-scale 4`
  completed both modes with 4/4 success. Greedy p50/p95 were
  `363.833/409.368 ms`; proportional p50/p95 were `575.914/634.549 ms`.
- Provider timing inspection for the scale campaign showed queue wait near zero
  in both modes. The current workload does not create sustained provider queue
  pressure, so proportional still mainly pays extra stage and dependency
  exchange overhead. The next campaign should use open-loop steady-state
  submission or a longer request window to create real provider backlog before
  claiming throughput benefits.
- Final checks: `git diff --check` passed, and
  `codegraph sync . && codegraph status .` reported the index is up to date.
