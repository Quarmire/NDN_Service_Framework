# Plan: Executable DI Layout Comparison

## Approach

Use the current NativeTracer execution plan and artifact set. The alternative
layout is expressed as an assignment change:

- baseline: four roles on four providers
- alternative: all four roles on one provider

The C++ provider executable already supports `--roles all|role,...`, so the
harness should group assignment rows by provider and launch one provider process
with a comma-separated role list.

## Evidence Contract

`planner-optimization.json` should mark `single-provider-serial` as executable
when the runtime layout is `single-provider-serial`. The default layout remains
`shared-backbone-current`.

## Validation

Run:

```bash
python3 -m py_compile \
  examples/python/NDNSF-DistributedInference/native_di_tracer/optimize_native_tracer_plan.py \
  examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
  Experiments/NDNSF_DI_NativeTracer_Minindn.py

./waf build --targets=di-native-provider,di-native-plan-schema-smoke,di-native-plan-manifest-smoke,di-native-provider-session-smoke

sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network --assignment default \
  --out /tmp/ndnsf-di-layout-default

sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network --assignment single-provider \
  --out /tmp/ndnsf-di-layout-single-provider
```
