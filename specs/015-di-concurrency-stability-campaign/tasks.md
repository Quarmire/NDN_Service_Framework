# Tasks: DI Concurrency Stability Campaign

- [x] S001 Create concurrency stability campaign spec, plan, and task list.
- [x] S002 Validate Python syntax for NativeTracer user/campaign scripts.
- [x] S003 Run `requests=2`, `concurrency=2`, 10x2 full-network MiniNDN campaign.
- [x] S004 Inspect `campaign-summary.json` and per-run CSV.
- [x] S005 Update spec/tasks with accepted artifact paths and interpretation.

## Result

Accepted artifacts:

- `/tmp/ndnsf-di-concurrency-campaign-10/campaign-summary.json`
- `/tmp/ndnsf-di-concurrency-campaign-10/campaign-runs.csv`

Both layouts completed all requests:

- `default` / `shared-backbone-current`: 20 success, 0 failure.
- `single-provider` / `single-provider-serial`: 20 success, 0 failure.

Per-request workload metrics:

- `default`: mean `478.957 ms`, p50 `446.470 ms`, p95 `511.443 ms`.
- `single-provider`: mean `567.271 ms`, p50 `537.988 ms`, p95 `596.554 ms`.

The stable concurrency-2 result supports the Feature 014 key-wrapping fix and
shows that shared-backbone becomes faster than single-provider when two
requests are outstanding.
