# Implementation Plan: Negative ACK Early Stop

## Summary

The smallest correct design is to keep negative ACK inside the existing ACK message. Providers already publish `RequestAckMessage` with `status=false`; the missing runtime behavior is on the user side. The user should remember negative ACKs, expose them in diagnostics, and fail early only when the complete known provider set has rejected the request.

## Design

### Wire Semantics

No new TLV type is needed.

```text
RequestAckMessage.status = false
RequestAckMessage.message = reason code, for example QUEUE_FULL or PROVIDER_BUSY
RequestAckMessage.payload = optional diagnostics, for example queue=10;gpu=busy
```

The reason code is intentionally carried in the existing message string. This avoids a message-format migration while giving tests and operators a stable string to inspect.

Recommended reason codes are exported by `NegativeAckReason.hpp`:

- `QUEUE_FULL`
- `PROVIDER_BUSY`
- `GPU_BUSY`
- `MODEL_UNAVAILABLE`
- `PERMISSION_DENIED`
- `UNSUPPORTED_REQUEST`
- `INTERNAL_ERROR`

Applications may still use other strings, but framework examples and tests should use the recommended set so diagnostics and experiment summaries remain comparable.

DI providers use the same codes when a provider is known but cannot accept the
request yet. Native readiness and Python artifact readiness use:

- `MODEL_UNAVAILABLE` while model/runtime artifacts are still installing;
- `INTERNAL_ERROR` when artifact/runtime provisioning has failed;
- `PROVIDER_BUSY`, `QUEUE_FULL`, or `GPU_BUSY` for admission or capacity
  rejection paths.

Python DI providers can opt into a provider-local admission policy that converts
existing `RuntimeTelemetryV1` fields into standard negative ACKs. This policy is
disabled unless the application passes it explicitly, so existing DI scheduling
and provider selection behavior remain unchanged by default. When enabled, the
policy may reject on aggregate queue depth, active worker count, low free GPU
memory, queue-wait EWMA, or unloaded model state, while keeping the telemetry
and threshold that caused the rejection in the ACK payload.

The native MiniNDN NativeTracer provider exposes the same concept through
provider-local environment variables injected by the harness. These variables
are only set when the experiment passes explicit admission flags. Native
readiness can reject with `PROVIDER_BUSY`, `QUEUE_FULL`, or `GPU_BUSY` based on
active workers, pending work, or advertised free memory. A small campaign script
runs baseline and admission-enabled cases and records p50/p95, throughput,
failure breakdown, and negative ACK reason counters.

The first NativeTracer admission campaign is intentionally conservative in its
interpretation. With the default one-provider-per-role layout, a provider-side
`PROVIDER_BUSY` rejection is observable, but it does not improve end-to-end
latency because there is no alternate provider for the rejected role. The
request still reaches the normal bounded timeout. This confirms the mechanism
and diagnostics, while showing that performance benefit requires either
role-level backup providers or planner support for retrying an alternate
runtime layout after a negative ACK.

The detailed provider state remains in the ACK payload as key/value diagnostics
such as `runtimeStatus=installing;negativeAckReason=MODEL_UNAVAILABLE;`.

### User Pending State

Each pending call keeps:

- `negativeAckProviders`: provider names that sent `status=false`;
- `negativeAckReasons`: provider URI to reason-code text.

The existing `requestAcks` list still stores all ACKs so custom selection can inspect both accepted and rejected candidates. Existing selection code already filters `status=true` before publishing selection.

### Early Stop Rule

After a negative ACK is recorded, the user checks whether the request was sent to an explicit provider list. If every explicit provider appears in `negativeAckProviders` and there are no successful ACK providers, the user marks the request as timed out and runs the normal timeout callback immediately.

This is deliberately conservative. Discovery-mode requests and learned-provider counts are not enough to prove that all possible providers rejected the request.

### Diagnostics

Add trace events:

- `NEGATIVE_ACK_RECORDED`
- `NEGATIVE_ACK_EARLY_STOP_ALL_KNOWN_PROVIDERS`

The normal timeout trace remains the terminal event, so benchmark tooling continues to see a failed request through the existing timeout path.

NativeTracer MiniNDN summaries aggregate negative ACK reason counters from both
user-side `NEGATIVE_ACK_RECORDED` events and provider-side native ACK decisions.
This lets DI campaigns report whether failures came from bounded-time delivery,
busy providers, missing models, or runtime errors.

The summary also includes a `failureBreakdown` object. It separates request
timeouts from negative ACK diagnostic signals so RPS campaigns can report both
the user-visible failure rate and the provider-side rejection causes.

## Validation

- Build the changed C++ examples/runtime.
- Run the existing mixed selective ACK regression.
- Run a new all-negative known-provider regression.
- Run one MiniNDN validation smoke after the regression tests.

## Risk

The main risk is accidentally treating a single negative ACK as terminal. The implementation avoids this by requiring an explicit known-provider set and by checking all providers in that set.
