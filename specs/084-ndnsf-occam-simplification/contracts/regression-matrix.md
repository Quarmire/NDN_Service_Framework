# Regression Matrix Contract

Commands marked `DISCOVER` must be replaced in the child baseline with the
current exact command before production edits. A description alone is not
executable evidence.

| Area | Baseline command/entry point | Required evidence | Blocking expectation |
|---|---|---|---|
| Core build/API | `./waf build && ./build/unit-tests` (child verifies exact target) | full build + generic API tests | all pass |
| Security | `examples/run_hello_auth_regression.sh`; `examples/run_nac_abe_attribute_routing_regression.sh`; `examples/run_token_handshake_negative_regression.sh` | permission, NAC-ABE, token/replay, bootstrap | all pass, no bypass |
| Normal/Targeted | `examples/run_hello_ack_payload_regression.sh`; `examples/run_selective_ack_custom_selection_regression.sh`; DISCOVER MiniNDN command | focused C++/Python tests + HELLO regressions | behavior unchanged |
| Collaboration | DISCOVER from current collaboration regression scripts | dependency and large-data tests | exact-name retrieval passes |
| DI | `python3 -m unittest discover -s tests/python -p 'test_ndnsf_di_*.py'` (child narrows exact list) | planner/runtime/cache/GUI tests | all default-path tests pass |
| DI MiniNDN | DISCOVER NativeTracer/Qwen canonical scripts | NativeTracer and Qwen smoke | execution succeeds coordinator-off |
| Repo storage | DISCOVER exact-packet/cache/restart commands | exact packet + cache + restart | wire bytes and SQLite authority preserved |
| Repo HA | DISCOVER canonical HA campaign command | quorum/failure/recovery/repair/catalog | 30/30 per run, W=2, valid replicas |
| UAV stream | DISCOVER stream/FEC unit commands | reorder/gap/stale/FEC/backlog | current semantics preserved |
| UAV MiniNDN | DISCOVER canonical live-video loss command | live video loss run | no new failure or unbounded backlog |
| Performance | child baseline exact commands under `experiment-gates.md` | matched p50/p95/completion | frozen threshold passes |

Every task that removes code cites one or more rows from this matrix.
