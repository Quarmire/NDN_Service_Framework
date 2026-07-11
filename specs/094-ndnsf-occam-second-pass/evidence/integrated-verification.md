# Integrated Verification

## Full regressions

- Python: `337 passed, 1 skipped` in 10.125 seconds.
- Core C++: `build/unit-tests --log_level=message` completed 214 test cases
  with no errors. Optional ONNX/generated-plan cases reported their documented
  skips.
- Full C++ build: `./waf build -j4` passed after migrating the UAV recording
  Repo to an explicit tiered SQLite store.
- Repo C++ smoke, tiered-cache, exact-packet, and HA executables passed.
- Repo Python: 90 tests passed after the convergence fixes.
- Native DI provider session smoke passed.

## Network checks

- Core HELLO MiniNDN: `PYTHON_HELLO_MININDN_OK` after removing the stale
  permission-token display from the example.
- Repo MiniNDN: `GENERIC_DISTRIBUTED_REPO_QUICK_MININDN_OK`, 53.5 seconds.
  The first two runs exposed a real fetch-routing defect: `FETCH_PREPARE`
  returned a stable forwarding hint, but the client discarded it and waited on
  the unadvertised object Data name. The repaired path advertises one stable
  per-repo locator, carries the returned hint and versioned Data name into the
  segment fetch, and no longer waits for per-object NLSR propagation.
- UAV quick check: `NDNSF_UAV_GUI_MININDN_QUICK_SMOKE_OK`.
- DI threaded MiniNDN: 60 requests over a 60-second open-loop window, 60
  successes, 0 failures, 0 timeouts, p50 210.42 ms, p95 1178.95 ms. All three
  NativeTracer roles executed. Dependency evidence recorded 120 successful
  publishes and 119 successful fetches.

DI evidence is under
`results/spec094-ndnsf-occam-second-pass/di-threaded-1rps/`; `summary.json`,
`planner-metrics.json`, and `dependency_object_counters.json` are the primary
result files. Local result output is not treated as source code.

## Scope and size

- Maintained non-test code from `74015ed` to the implementation head: 171
  additions, 1181 deletions, net -1010 lines. Specs, tests, docs, results, and
  agent files are excluded from this calculation.
- No file under `docs/PAPER`, `docs/proposal`, or proposal-defense paths changed.
- Final Occam recurrence guard: 0 active findings; 5 documentation, 210
  historical-spec, and 28 test-only references remain classified rather than
  reported as active code.
- Workspace size: 7.0 GiB. Filesystem free space: 8.0 GiB.

