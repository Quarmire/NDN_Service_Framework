# Spec 109 Audit Remediation Map

This checklist maps the 2026-07-12 adversarial audit to Revision 2 artifacts.

- [x] H1 baseline validity: `spec.md` FR-043/044, `plan.md` §§4/6, `MatchedBaselinePair`, CLI `staged-baseline`, T027/T098-T103.
- [x] H2 confounds/statistics: RQ1a/RQ1b, workload profile, confidence and percentile rules, T059/T101/T124/T143/T146.
- [x] H3 GPU truth: FR-017, `BackendObservation`, evidence Schema, semantic rules 8-10, T023/T078/T083/T097.
- [x] H4 fail-open contracts/ownership: keyed matrix, conditional evidence, source/predecessor Schemas, semantic validator, Spec 108 deployment binding.
- [x] H5 exact predecessors: Spec 107 T027/T028-T038 and Spec 108 T091-T102 named in requirements, plan, Schema, quickstart, tasks.
- [x] H6 source lineage: clean/sealed-dirty snapshot contract and tasks T001/T008.
- [x] M1 numerical equivalence: exact arrays plus hidden/KV/logit/top-1 checks.
- [x] M2 gate censoring: scoped gates and independent-size continuation.
- [x] M3 grouped cells: keyed immutable ledger and per-cell bundle closure.
- [x] M4 portability/audit separation: repository-local canonical CLI and post-implementation audit path.

Checked here means the finding is represented in the revised documents, not implemented or experimentally verified.
