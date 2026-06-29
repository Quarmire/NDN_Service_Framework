# Quickstart: DI Layout Latency Campaign

Run the full small campaign:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/run_layout_campaign.py \
  --runs 10 \
  --out-root /tmp/ndnsf-di-layout-campaign-10 \
  --provider-check-timeout 60
```

Inspect:

```bash
cat /tmp/ndnsf-di-layout-campaign-10/campaign-summary.json
cat /tmp/ndnsf-di-layout-campaign-10/campaign-runs.csv
```

The runner starts each MiniNDN harness run with `sudo -n` internally and cleans
Mininet state before each iteration. Run the campaign wrapper as the normal
user so it can pass the local Python packages needed by the harness into the
root child process.

Accepted 10x2 result:

- `default` / `shared-backbone-current`: mean `259.407 ms`, stddev `22.110 ms`, p50 `267.681 ms`, p95 `283.888 ms`.
- `single-provider` / `single-provider-serial`: mean `186.776 ms`, stddev `19.960 ms`, p50 `191.702 ms`, p95 `206.311 ms`.
- `single-provider` mean ratio: `0.7200`.
