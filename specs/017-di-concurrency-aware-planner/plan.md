# Plan: DI Concurrency-Aware Planner Evidence

## Approach

Keep runtime execution deterministic and assignment-driven, but make planner
evidence workload-aware.

The scoring model will estimate:

```text
criticalPathMs        role DAG critical path for one request
providerBottleneckMs  largest per-provider compute work per request
providerReadyQueuePressureMs
                      estimated provider ready-queue pressure from roles per
                      provider and workload concurrency
concurrencyQueueMs    estimated average queueing from outstanding requests
transferMs            dependency transfer plus fixed exchange overhead
totalEstimatedMs      criticalPathMs + concurrencyQueueMs + profile queue cost
```

The fixed exchange overhead is intentionally explicit in the network profile,
because the measured campaigns show that NDNSF dependency exchange has a real
fixed cost for small artifacts.

## Compatibility

`selectedCandidate` remains the forced runtime candidate requested by the
MiniNDN harness so previous experiments stay reproducible.

`plannerRecommendedCandidate` records the lowest estimated executable candidate
for the workload. This is the field to use for future automatic planner policy.

## Validation

1. Compile Python scripts. Complete.
2. Generate planner evidence for concurrency 1, 2, and 4. Complete.
3. Confirm the evidence carries the new fields. Complete.
4. Confirm recommendations match campaign direction. Complete:
   - concurrency 1: `single-provider-serial`
   - concurrency 2: `shared-backbone-current`
   - concurrency 4: `shared-backbone-current`
5. Calibrate provider queue-pressure fields against c4/c8 provider timing.
   Complete.

## Observed Result

The scoring model now separates the forced execution candidate from the planner
recommendation:

```text
selectedCandidate: forced by runtime assignment for reproducibility
plannerRecommendedCandidate: lowest estimated executable candidate
```

At `roleExecutionDelayMs=75`, the recommendation changes with workload
concurrency:

```text
concurrency=1 -> single-provider-serial
concurrency=2 -> shared-backbone-current
concurrency=4 -> shared-backbone-current
```

The model is intentionally simple and calibrated by existing campaigns. Its
ready-queue pressure fields now explain the measured high-concurrency result:
shared-backbone keeps `maxRolesPerProvider=1`, while single-provider puts four
roles on one provider and receives nonzero `providerReadyQueuePressureMs`.
