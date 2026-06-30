# Feature 038: DI Provider Metric Repeated Comparison

## Goal

Use the provider utilization metrics from feature 037 to explain the repeated
greedy vs proportional LLM MiniNDN results. Feature 036 showed proportional is
faster under 4 and 8 offered RPS. Feature 037 added provider-level metrics. This
feature reruns the repeated campaign and records whether the metrics explain the
latency and throughput difference.

## Design

- Keep the smallest Qwen NativeTracer artifacts.
- Use process-pool open-loop mode only.
- Run `greedy` and `proportional` at 4 and 8 offered RPS.
- Use 5 runs per mode/rate, matching feature 036.
- Compare:
  - submitted vs scheduled requests;
  - success rate;
  - observed success RPS;
  - p50/p95 latency;
  - provider estimated utilization;
  - provider queue wait and handler time;
  - role distribution across providers.

## Validation

- Run the repeated MiniNDN campaign.
- Parse aggregate `providerUtilization` from
  `llm-full-network-campaign-summary.json`.
- Record a compact table and interpretation in `tasks.md`.
- Run `git diff --check` and CodeGraph sync/status.

## Interpretation Rules

- If `submittedCount == scheduledRequestCount`, failures or lower observed RPS
  are not local driver admission artifacts.
- If greedy has one provider with much higher utilization and queue wait, while
  proportional spreads role events and lowers p50/p95, that supports the
  resource-aware planning argument.
- If proportional has better latency but higher dependency wait, record that
  tradeoff explicitly.
