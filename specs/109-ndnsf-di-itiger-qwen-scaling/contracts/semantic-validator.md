# Canonical semantic validator contract

`tools/ndnsf-di/validate_spec109.py` is the repository-local authority after JSON Schema validation. Personal Skills may invoke it but must not reimplement its decisions.

The validator must reject:

1. duplicate cell or run identities, a run pointing to a missing/different cell, or reuse across campaigns;
2. a finalized matrix containing `SUBMITTED`/`RUNNING`, an executed terminal cell without run/evidence, or a blocked/deferred cell without scoped gate identity;
3. source snapshots whose referenced diff/archive/content digests cannot be resolved and reproduced;
4. any missing, extra, failed, stale, schema-incompatible, or digest-mismatched required predecessor entry;
5. a Spec 109 profile that copies mutable deployment resources instead of resolving the Spec 108 profile/release digest;
6. candidate identity drift after first submission;
7. an overhead pair whose artifact/runtime/session/workload/cache/logging/stage/GPU/warmup/timeout/window fingerprint differs;
8. candidate correctness PASS without exact token arrays and every preregistered numerical checkpoint PASS;
9. GPU PASS with incomplete profiling, any model node on CPU/other provider, fallback use, or GPU UUID outside the Slurm allocation;
10. candidate performance PASS without a matched staged baseline PASS, three original 60-second repetitions, successful promotion, and valid authority lineage;
11. an available p50/p95/p99 below 20/100/1000 observations respectively, or an unavailable percentile with a numeric value;
12. propagation of a model-local gate to an unrelated model, or a systemic block without a shared dependency edge;
13. any authority upgrade to physical production or any secret/raw security-token content.

Validation output is deterministic JSON containing schema results, semantic rule IDs, input digests, `PASS/FAIL`, and no repaired data. The validator never submits, retries, edits, or deletes a cell.
