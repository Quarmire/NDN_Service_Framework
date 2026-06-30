# Feature 036: LLM Process-Pool Repeated Campaign

## Goal

Turn the process-pool driver result from a one-sample smoke into repeatable
MiniNDN evidence for high-concurrency NDNSF-DI planning. Feature 035 proved that
process-pool removes local user-driver backpressure. This feature uses it to run
repeated 4 and 8 RPS campaigns for `greedy` and `proportional`, then records
cross-run statistics that are suitable for design decisions.

## Design

- Keep the model fixed to the smallest Qwen NativeTracer artifacts.
- Use process-pool open-loop mode only; do not use threaded mode for evidence.
- Focus on 4 and 8 offered RPS because 1 and 2 RPS are mostly smoke points.
- Run multiple MiniNDN repetitions per mode/rate.
- Preserve result directories under `/tmp` and summarize the canonical JSON.
- Evaluate:
  - scheduled vs submitted requests;
  - success rate and local backpressure;
  - observed success RPS;
  - p50/p95 latency across runs;
  - layout allocation for greedy vs proportional.

## Validation

- Run the repeated MiniNDN campaign with process-pool mode.
- Parse `llm-full-network-campaign-summary.json` into a compact markdown table.
- Record whether proportional consistently improves high-rate behavior or if the
  one-run result from feature 035 is still not stable enough.
- Run `git diff --check` and CodeGraph sync/status after documentation updates.

## Interpretation Rules

- Treat 100% success with zero local backpressure as evidence that the driver is
  no longer clipping the workload.
- Treat observed success RPS below offered RPS as system service time/queueing,
  not driver admission failure, when submitted equals scheduled.
- Do not claim proportional is better unless repeated p50/p95 and success RPS
  support it at the same offered rate.
