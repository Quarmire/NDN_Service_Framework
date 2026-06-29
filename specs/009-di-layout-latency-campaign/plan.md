# Plan: DI Layout Latency Campaign

## Approach

Add a small Python campaign runner that wraps
`Experiments/NDNSF_DI_NativeTracer_Minindn.py`. Each run gets its own result
directory:

```text
<out-root>/<assignment>/run-NN
```

The runner parses each run's `summary.json`, extracts
`userExecution.elapsedMs`, and writes:

- `campaign-summary.json`
- `campaign-runs.csv`

## Statistics

For each assignment:

- `count`
- `meanMs`
- `stddevMs` using sample standard deviation
- `p50Ms` using median
- `p95Ms` using nearest-rank percentile
- `minMs`
- `maxMs`

## Validation

```bash
python3 -m py_compile \
  examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py

python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py \
  --runs 10 \
  --out-root /tmp/ndnsf-di-layout-campaign-10 \
  --provider-check-timeout 60
```
