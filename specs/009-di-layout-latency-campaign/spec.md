# Feature 009: DI Layout Latency Campaign

Status: Accepted

## Goal

Run a small repeated MiniNDN campaign for the two executable NativeTracer
layouts so the paper/slides can report p50, p95, mean, and standard deviation
instead of relying on one run.

The model remains the existing smallest Qwen-derived NativeTracer ONNX artifact
set.

## Scope

- Run `shared-backbone-current` through `--assignment default`.
- Run `single-provider-serial` through `--assignment single-provider`.
- Use the same full-network MiniNDN harness and topology for both layouts.
- Run 10 successful measured iterations per layout by default.
- Emit machine-readable JSON and CSV summary artifacts.

## Non-Goals

- Larger Qwen model artifacts.
- New layouts beyond the two already executable layouts.
- Statistical claims beyond a small validation campaign.

## Acceptance

- [x] Campaign runner can execute N runs per assignment.
- [x] Campaign output includes per-run elapsed latency and aggregate mean, stddev,
  p50, p95, min, and max.
- [x] A 10x2 campaign completes with all runs successful.
- [x] Spec records the accepted campaign paths and aggregate results.

## Accepted Campaign

Command:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py \
  --runs 10 \
  --out-root /tmp/ndnsf-di-layout-campaign-10 \
  --provider-check-timeout 60
```

Artifacts:

- Summary: `/tmp/ndnsf-di-layout-campaign-10/campaign-summary.json`
- Per-run CSV: `/tmp/ndnsf-di-layout-campaign-10/campaign-runs.csv`
- Per-run evidence: `/tmp/ndnsf-di-layout-campaign-10/{default,single-provider}/run-*`

Results:

| Assignment | Runtime candidate | Count | Mean ms | Stddev ms | p50 ms | p95 ms | Min ms | Max ms |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `default` | `shared-backbone-current` | 10 | 259.407 | 22.110 | 267.681 | 283.888 | 211.378 | 283.888 |
| `single-provider` | `single-provider-serial` | 10 | 186.776 | 19.960 | 191.702 | 206.311 | 143.871 | 206.311 |

Comparison:

- Mean delta: `-72.631 ms`
- p50 delta: `-75.979 ms`
- p95 delta: `-77.577 ms`
- Mean ratio: `0.7200`

Interpretation: under the current smallest Qwen NativeTracer artifact set,
`single-provider-serial` is faster than `shared-backbone-current` in this
small campaign. The likely reason is that this small model does not gain enough
parallelism from cross-provider splitting to pay for NDNSF dependency exchange;
the single-provider layout avoids that fixed exchange overhead while still
running the real ONNX NativeTracer path inside the same MiniNDN harness.
