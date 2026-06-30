# Feature 037: DI Provider Utilization Summary

## Goal

Explain why one LLM runtime layout is faster than another by exposing
provider-level execution, queueing, and utilization metrics in the MiniNDN
campaign summaries. Feature 036 showed that proportional splitting is
repeatably faster than greedy under the current tiny Qwen NativeTracer harness.
This feature makes the evidence explainable by reporting what each provider did.

## Design

- Reuse existing provider timing logs emitted by `NDNSF_DI_RUNTIME_TIMING=1`.
- Do not change proposal slides.
- Do not change the C++ execution path unless parser evidence proves the logs
  are insufficient.
- Add a parser in `Experiments/NDNSF_DI_NativeTracer_Minindn.py` that reads
  provider serve logs and summarizes:
  - executed role count;
  - unique session count;
  - queue wait;
  - input fetch wait;
  - runner/publish time;
  - handler time;
  - total time;
  - capacity snapshot maxima;
  - estimated busy utilization from handler time and observed execution window.
- Add `providerUtilization` to each run's `summary.json`.
- Add `providerUtilizationJson` and aggregate provider statistics to
  `run_llm_full_network_campaign.py` outputs.

## Validation

- Run Python compile checks for changed scripts.
- Run a small process-pool MiniNDN smoke that generates provider utilization.
- Verify the run summary contains provider metrics for greedy and proportional.
- Run `git diff --check` and CodeGraph sync/status.

## Interpretation Rules

- `handlerMs` estimates provider busy time for the role handler.
- `queueWaitMs` estimates time waiting behind other ready work on that provider.
- `inputFetchWaitMs` estimates time blocked after the worker starts while
  dependency inputs complete.
- `estimatedUtilization` is approximate: sum handler time divided by observed
  provider window and worker count. It is useful for relative diagnosis inside
  the same MiniNDN harness, not as a hardware benchmark.
