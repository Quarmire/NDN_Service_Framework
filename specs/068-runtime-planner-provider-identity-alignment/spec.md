# Feature Specification: Runtime Planner Provider Identity Alignment

## User Story

As an NDNSF-DI experiment runner, I want runtime-aware planner candidates to use
the same provider names as the actual MiniNDN assignment, so observed
provider-pair telemetry can match dependency edge-cost scoring.

## Requirements

- When `plan_tracer.py` receives `--provider-profiles-json`, runtime-aware
  metadata must be built with those provider names.
- The old fixture metadata remains the fallback when no profiles file exists.
- The policy summary must record whether runtime metadata came from provider
  profiles or fixtures.
- Existing planner behavior must remain available for unit fixtures.
- Regression coverage must verify provider-profile metadata uses real provider
  prefixes.

## Non-Goals

- Do not change provider launch assignment itself.
- Do not add new wire protocol fields.
- Do not infer runtime metadata from post-execution logs in this feature.

