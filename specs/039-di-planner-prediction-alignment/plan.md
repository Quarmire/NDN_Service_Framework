# Feature 039: DI Planner Prediction Alignment

## Goal

Make the LLM resource-aware planner emit machine-readable prediction evidence
that explains why a layout should scale better under concurrent requests. The
current measured evidence shows proportional planning lowers queueing compared
with greedy placement. This feature adds predicted provider load, queue risk,
and dependency-transfer cost so later campaigns can compare planner expectation
against MiniNDN measurements.

## Design

- Keep the smallest Qwen NativeTracer model and the existing greedy and
  proportional planner modes.
- Do not change runtime scheduling or provider execution behavior.
- Add optional planner inputs for offered load:
  - `--target-rps`: expected request arrival rate.
  - `--provider-workers`: per-provider worker slots for utilization estimates.
- For each provider, summarize:
  - assigned roles and layer count;
  - estimated per-request compute/service time;
  - dependency ingress/egress count and activation bytes;
  - predicted utilization when target RPS is provided;
  - queue risk class: idle, low, medium, high, saturated.
- Add plan-level prediction summary:
  - predicted bottleneck provider;
  - max predicted utilization;
  - dependency transfer count and total transfer MB;
  - whether the layout is compute-limited or transfer-limited.
- Preserve the existing plan JSON schema by adding new fields rather than
  renaming existing fields.

## Validation

- Compile the touched Python files.
- Generate greedy and proportional planner JSON at 8 offered RPS and inspect
  prediction fields.
- Run a short MiniNDN process-pool campaign for greedy and proportional to
  verify the prediction fields flow into `summary.json`, campaign CSV, and
  aggregate summary.
- Run `git diff --check` and CodeGraph sync/status.

## Interpretation Rules

- Predicted utilization is an explanatory estimate, not an admission-control
  guarantee.
- Queue risk should align qualitatively with provider metrics from MiniNDN:
  greedy should identify the 8GB provider as the bottleneck, while proportional
  should spread predicted load across the 2GB/4GB/8GB providers.
- If measured utilization differs from predicted utilization, record the
  mismatch instead of forcing the model to look correct.
