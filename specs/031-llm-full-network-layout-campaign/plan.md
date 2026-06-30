# Feature 031: LLM Full-Network Layout Campaign

## Goal

Measure greedy and proportional LLM role layouts through the real MiniNDN
full-network path. Feature 029 showed planner-derived capacity benefits, and
feature 030 proved the proportional LLM roles execute over MiniNDN. This
feature adds the small campaign needed to compare both planning modes with
network-measured latency, success rate, and throughput.

## Design

Add a campaign runner that invokes
`Experiments/NDNSF_DI_NativeTracer_Minindn.py` in full-network mode for each
planner mode:

```text
greedy
proportional
```

Each run uses the same service path `/Inference/NativeTracer`, but the policy
bundle is generated from the selected LLM planner mode. The campaign records:

- request count and concurrency;
- layer allocation;
- success and failure counts;
- mean, p50, p95, makespan, and throughput;
- result directory for detailed logs.

## Validation

- Compile the new campaign script.
- Run at least one full-network greedy/proportional comparison.
- Write CSV and JSON summary artifacts.
- Run `git diff --check` and CodeGraph sync/status.

## Interpretation

This campaign still uses deterministic LLM stage execution. It is valid for
comparing NDNSF full-network role assignment and dependency exchange overhead
under the same synthetic stage cost, but it is not yet a real ONNX LLM
throughput benchmark.
