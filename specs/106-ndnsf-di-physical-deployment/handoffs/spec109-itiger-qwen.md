# Spec 109 to Spec 106 physical-deployment handoff

Spec 109 does not grant physical-production authority. Its only accepted live
iTiger result is immutable 0.5B source-model transfer and sealing. No GPU
inference job ran because exact Spec 107/108 predecessors were incomplete.

Spec 106 may consume the following after those predecessor gates close:

- physical GPU performance on the selected production hardware, using a new
  candidate identity and original repetitions;
- real cross-host NFD/network behavior after Spec 108 T134 supplies admissible
  allocation-scoped network evidence;
- real UAV/camera/control integration and end-to-end operational behavior;
- production identity, trust-anchor, credential rotation, revocation, host
  hardening, and security operations;
- long-duration soak, resource leak, scheduler interruption, restart, recovery,
  and operator-response evidence.

It must not inherit PASS from the 0.5B storage result, offline backend tests, or
MiniNDN. The MiniNDN fake LLM pipeline currently preserves a `local deadline`
failure after the stale Python binding was rebuilt; resolve and requalify that
in the owning predecessor workflow before creating a physical candidate.

Inputs:

- Spec 109 release gate: `specs/109-ndnsf-di-itiger-qwen-scaling/release-gate.json`
- exact predecessor observation:
  `results/spec109-itiger-qwen/predecessor-gate.json`
- storage and model registry: `evidence/storage-verdict.md`
- capacity/placement gate: `evidence/large-model-capacity.md`
- complete matrix summary: `results/spec109-itiger-qwen/analysis/`

Physical runs require new Spec 106-owned identities, explicit authorization,
and their own rollback/soak acceptance. Spec 109 artifacts remain read-only
inputs and must not be rewritten from Spec 106.
