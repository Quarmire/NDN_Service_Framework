# Tasks: DI Layout Latency Campaign

- [x] C001 Add a NativeTracer layout campaign runner.
- [x] C002 Aggregate p50, p95, mean, stddev, min, and max.
- [x] C003 Emit JSON and CSV campaign artifacts.
- [x] C004 Validate syntax and one-run smoke campaign.
- [x] C005 Run 10x2 full-network MiniNDN campaign.
- [x] C006 Record accepted aggregate results in this spec.

## Validation

- `python3 -m py_compile examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py`
- `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py --runs 1 --out-root /tmp/ndnsf-di-layout-campaign-smoke2 --provider-check-timeout 60`
- `python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py --runs 10 --out-root /tmp/ndnsf-di-layout-campaign-10 --provider-check-timeout 60`

Accepted summary:

- `default`: mean `259.407 ms`, stddev `22.110 ms`, p50 `267.681 ms`, p95 `283.888 ms`.
- `single-provider`: mean `186.776 ms`, stddev `19.960 ms`, p50 `191.702 ms`, p95 `206.311 ms`.
- `single-provider` was faster by `72.631 ms` mean, `75.979 ms` p50, and `77.577 ms` p95.
