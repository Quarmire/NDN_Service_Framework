# Feature 018: DI Auto Runtime Assignment

Status: Accepted

## Goal

Turn the concurrency-aware planner recommendation from Feature 017 into an
executable MiniNDN harness mode.

`--assignment auto` should run planner evidence first, read
`plannerRecommendedCandidate`, and then execute the matching runtime assignment.

## Scope

- Add `auto` to `Experiments/NDNSF_DI_NativeTracer_Minindn.py --assignment`.
- Preserve existing fixed assignments:
  - `default`
  - `single-provider`
  - `alternate`
- Keep `selectedCandidate` as the actual runtime assignment used in the run.
- Record both:
  - requested assignment: `auto`
  - resolved assignment: `default` or `single-provider`
- Validate with local execution smoke for concurrency 1, 2, and 4.

## Non-Goals

- New runtime layout.
- Full multi-run campaign rerun.
- New planner cost model.
- Changing NDNSF service invocation.

## Acceptance

- [x] `--assignment auto --concurrency 1` resolves to `single-provider`.
- [x] `--assignment auto --concurrency 2` resolves to `default`.
- [x] `--assignment auto --concurrency 4` resolves to `default`.
- [x] Summary records requested assignment, resolved assignment,
  selected candidate, and planner recommendation.
- [x] Existing fixed assignments still work.
- [x] Minimal full-network smoke confirms `auto` drives the executable runtime
  assignment, not only local policy generation.

## Evidence Commands

```bash
python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --local-execution-only \
  --assignment auto \
  --role-execution-delay-ms 75 \
  --requests 1 \
  --concurrency 1 \
  --out /tmp/ndnsf-di-auto-c1

python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --local-execution-only \
  --assignment auto \
  --role-execution-delay-ms 75 \
  --requests 4 \
  --concurrency 4 \
  --out /tmp/ndnsf-di-auto-c4
```

## Accepted Evidence

Local auto smoke results:

| Path | Requests | Concurrency | assignmentRequested | assignmentResolved | selectedCandidate | plannerRecommendedCandidate |
| --- | ---: | ---: | --- | --- | --- | --- |
| `/tmp/ndnsf-di-auto-c1/summary.json` | 1 | 1 | `auto` | `single-provider` | `single-provider-serial` | `single-provider-serial` |
| `/tmp/ndnsf-di-auto-c2/summary.json` | 2 | 2 | `auto` | `default` | `shared-backbone-current` | `shared-backbone-current` |
| `/tmp/ndnsf-di-auto-c4/summary.json` | 4 | 4 | `auto` | `default` | `shared-backbone-current` | `shared-backbone-current` |

Fixed-assignment compatibility:

- `/tmp/ndnsf-di-fixed-default-smoke/summary.json`: `default` remained
  `shared-backbone-current`.
- `/tmp/ndnsf-di-fixed-single-smoke/summary.json`: `single-provider` remained
  `single-provider-serial`.

Full-network auto smoke results:

| Path | Requests | Concurrency | assignmentResolved | selectedCandidate | userExecution | dependencyExecution |
| --- | ---: | ---: | --- | --- | --- | --- |
| `/tmp/ndnsf-di-full-auto-c1/summary.json` | 1 | 1 | `single-provider` | `single-provider-serial` | `executed` | `executed` |
| `/tmp/ndnsf-di-full-auto-c2/summary.json` | 2 | 2 | `default` | `shared-backbone-current` | `executed` | `executed` |
| `/tmp/ndnsf-di-full-auto-c4/summary.json` | 4 | 4 | `default` | `shared-backbone-current` | `executed` | `executed` |

Implementation note: `auto` uses a probe policy bundle to choose the runtime
assignment, then regenerates the final policy bundle for the resolved runtime.
The summary keeps the auto decision in `plannerRecommendedCandidate` and records
the final bundle's own recommendation as `finalPolicyRecommendedCandidate`.
