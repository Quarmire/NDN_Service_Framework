# Plan Lease and Attempt Contract

## Plan Validation Result

Every reuse decision returns:

```json
{
  "planId": "...",
  "decision": "reuse|replan|defer|reject",
  "checkedAtMs": 0,
  "predicates": [
    {"name": "telemetry-fresh", "status": "pass|fail", "observed": 100, "limit": 2000}
  ],
  "selectedProviders": [],
  "rejectedProviders": [{"provider": "...", "reason": "stale-telemetry"}]
}
```

Unknown mandatory predicates fail. No score can override a failed feasibility,
security, evidence, or freshness predicate.

## Attempt Identity

All role assignments and dependency object names carry:

```text
requestId
attemptEpoch
planId
role
scope
```

Only epochs `0` and `1` are valid in the pilot. Epoch 1 requires a recorded
terminal/superseded transition for epoch 0. Provider responses echo the attempt
epoch inside the authenticated DI payload. Core wire names are unchanged.

Terminal reasons are stable machine-readable codes:

```text
PROVIDER_LOST
STRAGGLER_DEADLINE
DEPENDENCY_MISSING
DEPENDENCY_HASH_MISMATCH
PLAN_STALE
TELEMETRY_STALE
CACHE_MISS_FULL_CONTEXT_REQUIRED
ATTEMPT_CANCELLED
NO_COMPATIBLE_REPLACEMENT
REQUEST_DEADLINE
```
