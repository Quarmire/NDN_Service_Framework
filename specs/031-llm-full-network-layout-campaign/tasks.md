# Tasks: LLM Full-Network Layout Campaign

## Phase 1: Planning Artifacts

- [x] T001 Create feature plan in `specs/031-llm-full-network-layout-campaign/plan.md`.
- [x] T002 Create task list in `specs/031-llm-full-network-layout-campaign/tasks.md`.

## Phase 2: Campaign Support

- [x] T003 Add planner mode and assignment-label support to the LLM bundle generator.
- [x] T004 Add `--llm-planner-mode` support to the MiniNDN harness.
- [x] T005 Add `run_llm_full_network_campaign.py` to run greedy/proportional full-network comparisons.

## Phase 3: Verification

- [x] T006 Compile changed Python scripts.
- [x] T007 Run one greedy/proportional MiniNDN full-network campaign.
- [x] T008 Verify CSV/JSON campaign outputs include success counts, p50/p95, throughput, and layer allocation.
- [x] T009 Run `git diff --check` and CodeGraph sync/status.

## Evidence

- Python compile passed:
  `PYTHONPYCACHEPREFIX=/tmp/ndnsf_pycache python3 -m py_compile ...`
- C++ manifest smoke rebuild passed after allowing single-stage plans without
  dependency output:
  `./waf build --targets=di-native-plan-manifest-smoke -j4`.
- Minimal full-network campaign:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_llm_full_network_campaign.py --out-root /tmp/ndnsf-llm-layout-campaign --runs 1 --workloads c1:1:1 --modes greedy,proportional`
  wrote `/tmp/ndnsf-llm-layout-campaign/llm-full-network-campaign-summary.json`.
  Greedy completed 1/1 with p50/p95 `160.709 ms` and allocation
  `{"/NDNSF-DI/Tracer/provider/llm-8gb": 28}`. Proportional completed 1/1
  with p50/p95 `254.145 ms` and allocation
  `{"/NDNSF-DI/Tracer/provider/llm-2gb": 4, "/NDNSF-DI/Tracer/provider/llm-4gb": 8, "/NDNSF-DI/Tracer/provider/llm-8gb": 16}`.
- c2 full-network campaign:
  `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_llm_full_network_campaign.py --out-root /tmp/ndnsf-llm-layout-campaign-c2 --runs 1 --workloads c2:4:2 --modes greedy,proportional`
  wrote `/tmp/ndnsf-llm-layout-campaign-c2/llm-full-network-campaign-summary.json`.
  Greedy completed 4/4 with p50 `132.789 ms`, p95 `201.041 ms`.
  Proportional completed 4/4 with p50 `192.964 ms`, p95 `253.260 ms`.
- Interpretation: with the current deterministic tiny-Qwen workload, greedy
  remains faster because it avoids cross-provider dependency exchange. The
  proportional layout is now measured end-to-end, but it does not yet improve
  latency for this small synthetic workload.
- Final checks: `git diff --check` passed, and
  `codegraph sync . && codegraph status .` reported the index is up to date.
