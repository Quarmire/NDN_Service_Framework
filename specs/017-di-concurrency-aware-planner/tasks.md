# Tasks: DI Concurrency-Aware Planner Evidence

- [x] R001 Create Feature 017 spec, plan, and task list.
- [x] R002 Add workload-aware cost fields to NativeTracer optimizer evidence.
- [x] R003 Pass workload concurrency from plan generation and MiniNDN harness.
- [x] R004 Validate Python syntax.
- [x] R005 Generate evidence for concurrency 1, 2, and 4.
- [x] R006 Record recommendations and remaining limits.

## Result

Accepted evidence:

- `/tmp/ndnsf-di-planner-c1/planner-optimization.json`
- `/tmp/ndnsf-di-planner-c2/planner-optimization.json`
- `/tmp/ndnsf-di-planner-c4/planner-optimization.json`
- `/tmp/ndnsf-di-planner-single-c4/planner-optimization.json`
- `/tmp/ndnsf-di-planner-harness-c4/summary.json`

Recommendations:

- `concurrency=1`: `single-provider-serial`
- `concurrency=2`: `shared-backbone-current`
- `concurrency=4`: `shared-backbone-current`

The harness smoke confirmed `workloadConcurrency=4` is propagated into
`optimizationEvidence`.
