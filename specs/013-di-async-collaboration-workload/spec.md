# Feature 013: DI Async Collaboration Workload

Status: Boundary identified

## Goal

Enable true outstanding NativeTracer collaboration requests from one user
process, then measure shared-backbone versus single-provider under provider
capacity pressure using makespan, p95 latency, throughput, and success count.

## Scope

- Add an async Python binding for `ServiceUser::RequestCollaboration`.
- Add a Python `ServiceUser.request_collaboration_async()` wrapper.
- Extend `user_driver.py` with `--concurrency` while preserving closed-loop
  default behavior.
- Thread concurrency through the MiniNDN harness and campaign runner.
- Run a small full-network async workload campaign at the current 75 ms per-role
  capacity pressure.

## Non-Goals

- New NDNSF wire messages.
- Provider runtime redesign.
- Changing the smallest Qwen NativeTracer artifacts.
- Replacing closed-loop workload support.

## Acceptance

- [x] Python binding can submit collaboration requests asynchronously and invoke
  Python response/timeout callbacks.
- [x] `user_driver.py --requests N --concurrency C` can create outstanding
  collaboration attempts through child requester processes with isolated
  requester identities and copied PIB state.
- [x] Existing `--concurrency 1` closed-loop behavior remains compatible.
- [x] MiniNDN summary records concurrency and workload metrics.
- [x] Campaign runner records concurrency.
- [ ] Full-network concurrent smoke completes with all requests successful.
- [x] Results and next recommendation are recorded here.

## Evidence

- Sequential sanity passed:
  `/tmp/ndnsf-di-sync-sanity`.
- Concurrent same-process async and threaded attempts did not produce reliable
  completion.
- Child-process requesters required unique requester identities, worker
  user-policies, copied child HOME/.ndn directories to avoid PIB locks, and a
  running parent Face to serve pre-published scope-key large data.
- With those fixes, `requests=2`, `concurrency=2` still completed only one
  request:
  `/tmp/ndnsf-di-childhome-workerid-smoke-c2b`,
  `/tmp/ndnsf-di-childhome-workerid-stagger-c2`, and
  `/tmp/ndnsf-di-childhome-workerid-smoke-c2-longtimeout`.
- Core trace in `/tmp/ndnsf-di-childhome-workerid-trace-c2c` shows incomplete
  or late ACK delivery before selection. Requests time out with
  `no_selection_published`, even with longer ACK/request windows.

## Recommendation

Do not expand the layout campaign until the service invocation layer can handle
bounded-time outstanding collaboration requests. The next feature should target
ACK/selection fairness and SVS delivery for multiple simultaneous requester
identities in the same service group.
