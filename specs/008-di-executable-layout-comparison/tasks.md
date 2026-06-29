# Tasks: Executable DI Layout Comparison

- [x] L001 Add `single-provider` assignment to the NativeTracer MiniNDN harness.
- [x] L002 Group provider launches by provider/node and pass comma-separated
  role lists.
- [x] L003 De-duplicate identity bootstrap for repeated provider identities.
- [x] L004 Let policy generation mark the active runtime candidate.
- [x] L005 Record active layout and selected candidate in harness summaries.
- [x] L006 Run baseline and single-provider full-network MiniNDN validation.
- [x] L007 Record measured latency comparison and accepted evidence.

## Validation

```bash
python3 -m py_compile \
  examples/python/NDNSF-DistributedInference/native_di_tracer/optimize_native_tracer_plan.py \
  examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
  examples/python/NDNSF-DistributedInference/native_di_tracer/compare_layout_results.py \
  Experiments/NDNSF_DI_NativeTracer_Minindn.py

PYTHONPATH=NDNSF-DistributedInference:pythonWrapper \
  python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
  --out /tmp/ndnsf-di-layout-policy-default \
  --summary-json /tmp/ndnsf-di-layout-policy-default-summary.json \
  --runtime-candidate shared-backbone-current

PYTHONPATH=NDNSF-DistributedInference:pythonWrapper \
  python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
  --out /tmp/ndnsf-di-layout-policy-single \
  --summary-json /tmp/ndnsf-di-layout-policy-single-summary.json \
  --runtime-candidate single-provider-serial

./waf build --targets=di-native-provider,di-native-plan-schema-smoke,di-native-plan-manifest-smoke,di-native-provider-session-smoke

python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --quick-smoke

python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --local-execution-only \
  --assignment single-provider \
  --out /tmp/ndnsf-di-layout-single-local

sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network \
  --assignment single-provider \
  --out /tmp/ndnsf-di-layout-single-provider \
  --provider-check-timeout 60

sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network \
  --assignment default \
  --out /tmp/ndnsf-di-layout-default \
  --provider-check-timeout 60

python3 examples/python/NDNSF-DistributedInference/native_di_tracer/compare_layout_results.py \
  --baseline /tmp/ndnsf-di-layout-default \
  --alternative /tmp/ndnsf-di-layout-single-provider \
  --out-json /tmp/ndnsf-di-layout-comparison.json \
  --out-csv /tmp/ndnsf-di-layout-comparison.csv
```

Accepted result:

```text
baseline: shared-backbone-current 236.82666099921335 ms
alternative: single-provider-serial 179.01252299998305 ms
deltaMs: -57.814
```
