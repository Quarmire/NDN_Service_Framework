# Feature Specification: NativeTracer Provider-Pair Probe

## User Story

As an NDNSF-DI experiment runner, I want NativeTracer full-network MiniNDN runs
to actively measure dependency-edge provider RTTs before the user workload, so
the run can emit `providerPairTelemetry` evidence that later planner runs can
consume.

## Requirements

- The probe must run after MiniNDN routing and provider provisioning are ready.
- The probe must run before the NativeTracer user driver starts.
- For each dependency edge in the native execution plan, run ndnping from the
  consumer node to the producer provider prefix and write
  `dependency-edge-ndnping-rtt-stats.json`.
- The summary must record probe status separately from collected telemetry.
- Probe failure must not fail the whole experiment; it should be recorded as
  `providerPairTelemetryProbe.status=failed`.
- A CLI/profile option must allow skipping the probe for fast smoke runs.
- Pure helper tests must cover ndnping parsing, provider metadata extraction,
  and dependency edge loading.

## Non-Goals

- Do not change NDNSF security or service invocation.
- Do not make the current run's post-probe telemetry change the plan already
  generated for that same run.
- Do not add bandwidth probing in this feature.

