# Plan: DI Auto Runtime Assignment

## Approach

The harness will use a two-step planner flow for `--assignment auto`:

1. Generate a temporary policy bundle with `runtime-candidate=shared-backbone-current`
   and the requested workload concurrency.
2. Read `plannerRecommendedCandidate`.
3. Resolve to:
   - `single-provider` when recommendation is `single-provider-serial`
   - `default` when recommendation is `shared-backbone-current`
4. Regenerate the final policy bundle for the resolved runtime candidate.

This keeps fixed-assignment reproducibility while allowing `auto` to be driven
by planner evidence.

## Validation

- Python syntax validation. Complete.
- Local execution smoke for `auto` at concurrency 1, 2, and 4. Complete.
- Minimal full-network smoke for `auto` at concurrency 1, 2, and 4. Complete.
- Check `summary.json` for:
  - `assignmentRequested`
  - `assignmentResolved`
  - `optimizationEvidence.selectedCandidate`
  - `optimizationEvidence.plannerRecommendedCandidate`
  Complete.

## Expected Mapping

```text
concurrency=1 -> single-provider
concurrency=2 -> default
concurrency=4 -> default
```

## Observed Result

The observed mapping matched the expected mapping:

```text
c1: auto -> single-provider -> single-provider-serial
c2: auto -> default -> shared-backbone-current
c4: auto -> default -> shared-backbone-current
```

The fixed `default` and `single-provider` modes still run through local
execution successfully.

The same mapping also executed in MiniNDN full-network mode:

```text
c1: userExecution=executed, dependencyExecution=executed
c2: userExecution=executed, dependencyExecution=executed
c4: userExecution=executed, dependencyExecution=executed
```
