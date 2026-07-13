# Qwen scaling report

## Result

No controlled Qwen scaling or NDNSF-DI overhead claim is available. The full
matrix contains 105/105 represented cells and all are terminal `BLOCKED`; no
cell has a Slurm run ID or promoted measurement digest.

The systemic cause is the exact predecessor gate: Spec 107 T027/T028–T038 and
Spec 108 T091–T102 remain incomplete. The independent 72B quota gate and
multi-node network gate add model-local and placement-local constraints without
changing unrelated cell identities.

## Analysis boundaries

- `analysis/cells.csv` retains the complete denominator; it does not filter to
  successful cells.
- `analysis/matrix-summary.json` reports all terminal states per model.
- `analysis/matched-overhead.json` is `UNAVAILABLE` because no accepted staged
  baseline/candidate pair exists.
- Full-model Transformers timing is excluded from overhead by construction.
- `small-medium-scaling.csv` is descriptive status only. The controlled
  common-hardware subset is empty.
- Reproduction is `INCONCLUSIVE` and submitted zero jobs because selecting a
  non-accepted cell would create false authority.

The sealed 0.5B source model is a storage result only. No throughput, TTFT,
inter-token, percentile, confidence-interval, CUDA-backend, or candidate scaling
number may be inferred from this campaign state.
