# Implementation Status

## Completed Isolation

- Default package exports no advisory or semantic-cache symbols.
- Advisory implementation, wire service, CLI switch, policies, and comparison
  branch are deleted after the frozen gate failed.
- Semantic-cache implementation physically resides in
  `experimental/semantic_cache/implementation.py`.
- `runtime_v1.py` retains provider-local Exact Forward Cache but contains no
  advisory or semantic-cache implementation.
- Default planner registry contains executable handlers only.
- Retry decisions use typed reasons and explicit idempotency.
- The unused Merge deployment ref-count authority is removed.

## Local Regression Evidence

```text
policy isolation                    7/7 PASS
runtime v1                         22/22 PASS
runtime-aware campaign             24/24 PASS
runtime-aware planner              10/10 PASS
DI scenarios                       42/42 PASS
execution lease suites             21/21 PASS
semantic cache demo                 2/2 PASS
semantic llama provider (no socket) 4/4 PASS
headless GUI                       20/20 PASS
```

The final combined DI Python run passed 330 tests with one expected display
skip after deleting the advisory-only test surface.
Full C++ Core passed 215/215 and all six security regressions passed after the
same working-tree changes.

## MiniNDN Acceptance

The coordinator-off Qwen ONNX NativeTracer capacity-pool smoke at
`results/spec087-advisory-smoke/pure/rps-0p2/summary.json` completed 2/2
multi-user requests with p50 324.35 ms and p95 332.64 ms. This run used the
real NDNSF wire path and provider-owned admission leases.

## Frozen Advisory Decision

The ten matched pairs under `results/spec087-advisory-gate/` failed the frozen
retention gate. Mean lease-conflict rate increased from 0.06615 to 0.10191
(relative improvement -54.06%), completion fell from 70.0% to 52.5%, and p95
rose from 5514.96 ms to 5640.08 ms. The paired 95% bootstrap interval
[-0.07136, 0.00387] crosses zero. Advisory coordination is therefore deleted,
not tuned or reinterpreted.
