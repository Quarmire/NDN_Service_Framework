# Feature 034: LLM Open-Loop RPS Sweep

## Goal

Run sustained open-loop NDNSF-DI LLM full-network comparisons across multiple
offered rates. Feature 033 proved that fixed-rate submission works for a short
smoke; this feature makes the campaign runner produce rate-labeled greedy vs
proportional results so we can see when provider backlog, local backpressure, or
failures appear.

## Design

- Add `--target-rps-series` to the LLM full-network campaign runner.
- Keep `--target-rps` as the single-rate default.
- For open-loop runs, automatically raise the per-run request cap to
  `ceil(targetRps * openLoopDurationS)` when the workload request count is
  smaller. This prevents high-rate sweeps from silently submitting too few
  requests.
- Include the rate in the workload label and result directory so summaries can
  compare `greedy/base-r4` and `proportional/base-r4` directly.
- Preserve the existing one-rate behavior for current scripts and evidence.

## Validation

- Compile changed Python scripts.
- Run a short two-rate MiniNDN smoke.
- If the smoke passes, run a small 1/2/4/8 RPS campaign with short duration to
  capture first-order behavior.
- Record result paths and interpretation in `tasks.md`.

## Interpretation

This is still a synthetic NativeTracer/Qwen-tiny style campaign. Because the
current robust open-loop path uses one child ServiceUser per submitted request,
latency includes process startup overhead. The useful evidence is relative
trend under the same harness: success/failure, local backpressure, p50/p95, and
whether proportional placement starts helping as offered load rises.
