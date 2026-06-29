# Quickstart: Executable DI Layout Comparison

Build prerequisites:

```bash
./waf configure --with-examples
./waf build --targets=di-native-provider,di-native-plan-schema-smoke,di-native-plan-manifest-smoke,di-native-provider-session-smoke
```

Run baseline:

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network \
  --assignment default \
  --out /tmp/ndnsf-di-layout-default
```

Run alternative:

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network \
  --assignment single-provider \
  --out /tmp/ndnsf-di-layout-single-provider
```

Compare:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/compare_layout_results.py \
  --baseline /tmp/ndnsf-di-layout-default \
  --alternative /tmp/ndnsf-di-layout-single-provider \
  --out-json /tmp/ndnsf-di-layout-comparison.json \
  --out-csv /tmp/ndnsf-di-layout-comparison.csv
```
