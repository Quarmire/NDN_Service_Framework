# Feature Specification: DI Provider-Pair Matrix Planner Input

## User Story

As an NDNSF-DI experiment runner, I want the NativeTracer planner to consume a
core `ProviderNetworkMatrix` captured from prior provider-pair telemetry, so
dependency-heavy role placement can account for observed provider-to-provider
network cost.

## Requirements

- The existing default runtime-aware planner behavior must remain unchanged
  when no matrix file is provided.
- The plan tracer must accept an optional matrix JSON input.
- The input may be either a raw `ProviderNetworkMatrix` payload or a previous
  NativeTracer summary containing `providerPairTelemetry.matrix`.
- The MiniNDN harness must expose and pass the same option to the plan tracer.
- The generated policy summary must record the matrix source and metric count.
- Regression tests must cover argument plumbing and summary-wrapper loading.

## Non-Goals

- Do not change provider runtime, security, permission, or wire protocol.
- Do not make same-run post-execution telemetry affect the already-started
  plan. Observed telemetry is reusable by later runs or campaign phases.
- Do not replace the existing fixture matrix; keep it as the default fallback.

