# Feature Specification: Provider-Pair Telemetry Collection

## User Story

As an NDNSF-DI experiment runner, I want NativeTracer MiniNDN summaries to expose
provider-to-provider network measurements using the reusable NDNSF core
telemetry vocabulary, so later planners and papers can reason about dependency
exchange cost without inventing DI-only metric fields.

## Requirements

- The NativeTracer harness must preserve existing experiment behavior and
  planner choices.
- When dependency-edge ndnping evidence exists, the harness must convert it into
  core `PeerNetworkMetric` records.
- Metrics must use dependency dataflow direction: producer provider as
  `src_peer`, consumer provider as `dst_peer`.
- Summary JSON must include a stable `providerPairTelemetry` section with
  status, source, metric count, metrics, and a `ProviderNetworkMatrix` payload.
- When evidence is absent or malformed, summary JSON must report
  `not-available` without failing the experiment.
- Unit-level regression coverage must verify a fixture can be collected and
  consumed through `ProviderNetworkMatrix`.

## Non-Goals

- Do not change runtime-aware planning policy in this feature.
- Do not add active bandwidth probing.
- Do not move DI dependency semantics into core.

