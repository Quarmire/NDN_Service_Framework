# Plan: DI Optimization Evidence Roadmap

## Design

Add an optimization-evidence layer around the existing NativeTracer plan rather
than changing the model or runtime. The existing generated files remain the
canonical executable plan:

- `native-execution-plan.json`
- `service-manifest.json`

A new script generates:

- `planner-optimization.json`
- `planner-optimization.csv`

The JSON file is the durable contract. The CSV file is a quick comparison table
for papers, slides, and experiment inspection.

## Candidate Model

The optimizer evaluates the current four-role graph:

- `/Backbone`
- `/Head/Shard/0`
- `/Head/Shard/1`
- `/Merge`

It scores several layouts:

- current shared-backbone multi-provider plan
- single-provider serial plan
- replicated-backbone parallel plan
- pipeline-stages plan
- multi-provider merge-centered plan

Only layouts supported by the current NativeTracer runtime can be selected for
execution. Other layouts are recorded as estimated alternatives.

## Cost Model

Each candidate receives:

- compute cost: role runtime adjusted by provider compute score
- transfer cost: expected dependency bytes over the provider-pair network path
- queue/load cost: provider queue depth penalty
- total estimated latency

The model is intentionally small and deterministic. It provides planner
evidence, not a claim of final optimal scheduling.

## Validation

Run:

```bash
python3 -m py_compile \
  examples/python/NDNSF-DistributedInference/native_di_tracer/optimize_native_tracer_plan.py \
  examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
  Experiments/NDNSF_DI_NativeTracer_Minindn.py

PYTHONPATH=NDNSF-DistributedInference:pythonWrapper \
  python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
  --out /tmp/ndnsf-di-optimization-policy \
  --summary-json /tmp/ndnsf-di-optimization-policy-summary.json
```
