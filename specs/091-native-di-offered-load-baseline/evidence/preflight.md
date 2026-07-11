# Screening Preflight

**Runtime source commit**: `21b343d`
**Date**: 2026-07-11

## Frozen Command

For each `MODE` in `child`, `threaded`, and `process-pool`:

```bash
sudo -n timeout 240s python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --runtime-profile examples/di-native-tracer.runtime.json \
  --out results/spec091-native-di-offered-load-baseline/MODE \
  --requests 60 --concurrency 4 --target-rps 1 \
  --open-loop-duration-s 60 --open-loop-driver-mode MODE \
  --provider-check-timeout 60 --no-local-execution-only --full-network \
  --skip-provider-pair-telemetry-probe
```

The timeout is an outer monitor only; the harness computes its own workload and
request deadlines. The runtime profile fixes Qwen proportional planning,
AI_Lab topology, two-user fixture, and real full-network execution.

## Thresholds

- scheduling-capable: submitted/scheduled >= 0.95, maximum schedule slip below
  1,000 ms, and zero local backpressure failures;
- stable: success >= 0.99, achieved/offered RPS >= 0.95, dependency execution
  complete, and no malformed/security error.

## Dry-Run Result

All three dry runs resolved to requests 60, concurrency 4, target 1 RPS, Qwen
proportional assignment, enabled runtime-aware user planning, the same
multi-user fixture, and disabled provider-pair probe. Differences were limited
to the output/planner-metrics paths and `--open-loop-driver-mode MODE`.

Process inspection found no stale MiniNDN, provider, or user driver before the
screening commands. The worktree contained only Spec 091 documents; no runtime
source was dirty. Host capacity was 9.3 GiB available memory and 8.4 GiB free
disk, sufficient for bounded text/JSON results.

## Experiment Integrity

This is one deterministic screening run per mode. No confidence interval,
maximum stable RPS, or paper-facing superiority claim is permitted from these
three runs.

