# Tasks: DI Concurrency-4 Scaling Campaign

- [x] Q001 Create concurrency-4 scaling campaign spec, plan, and task list.
- [x] Q002 Validate Python syntax for NativeTracer user/campaign scripts.
- [x] Q003 Run `requests=4`, `concurrency=4`, 10x2 full-network MiniNDN campaign.
- [x] Q004 Inspect and aggregate campaign artifacts.
- [x] Q005 Update spec/tasks with accepted paths, metrics, and interpretation.

## Result

Accepted artifacts:

- `/tmp/ndnsf-di-concurrency4-campaign-10/campaign-summary.json`
- `/tmp/ndnsf-di-concurrency4-campaign-10/campaign-runs.csv`

Both layouts completed all requests:

- `default` / `shared-backbone-current`: 40 success, 0 failure.
- `single-provider` / `single-provider-serial`: 40 success, 0 failure.

Per-request workload metrics:

- `default`: mean `458.414 ms`, p50 `441.790 ms`, p95 `510.167 ms`.
- `single-provider`: mean `619.563 ms`, p50 `588.057 ms`, p95 `709.669 ms`.

`shared-backbone-current` remained faster at concurrency 4, strengthening the
case for concurrency-aware planner scoring.
