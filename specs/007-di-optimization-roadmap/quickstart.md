# Quickstart: DI Optimization Evidence

Generate the policy bundle and optimization evidence:

```bash
PYTHONPATH=NDNSF-DistributedInference:pythonWrapper \
  python3 examples/python/NDNSF-DistributedInference/native_di_tracer/plan_tracer.py \
  --out /tmp/ndnsf-di-optimization-policy \
  --summary-json /tmp/ndnsf-di-optimization-policy-summary.json
```

Inspect:

```bash
cat /tmp/ndnsf-di-optimization-policy/planner-optimization.json
cat /tmp/ndnsf-di-optimization-policy/planner-optimization.csv
cat /tmp/ndnsf-di-optimization-policy-summary.json
```

Expected:

```text
contractVersion: di-plan-v2
sourceModel: Qwen/Qwen2.5-0.5B-Instruct
modelUnchanged: true
selectedCandidate.id: shared-backbone-current
```
