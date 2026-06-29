# Feature 014: ACK Selection Concurrency Fairness

Status: Complete

## Goal

Make NDNSF service invocation handle multiple outstanding collaboration requests
against the same provider set without starving one request before selection.

The immediate regression target is the NativeTracer concurrent diagnostic:
`requests=2`, `concurrency=2`, 75 ms role delay, default shared-backbone layout.
Both requests should collect enough role ACKs, publish selection, enter provider
execution, and return responses within bounded time.

## Scope

- Fix ACK matching and ACK duplicate/replay handling so different requester
  identities and request IDs do not interfere.
- Fix selection scheduling so a collaboration request does not permanently miss
  selection when ACKs arrive slightly late or in bursts.
- Preserve existing sequential behavior and token checks.
- Keep all recovery in NDNSF/SVS/service invocation; do not change planner
  scoring or Qwen NativeTracer artifacts.

## Non-Goals

- New wire message formats unless existing state handling cannot represent the
  fix.
- Provider runtime redesign.
- Larger models.
- Slide/paper edits.

## Acceptance

- [x] `requests=1`, `concurrency=1` full-network NativeTracer still passes.
- [x] `requests=2`, `concurrency=2` full-network NativeTracer passes with both
  requests successful.
- [x] Provider logs show handler timing for both request sessions across all
  required roles.
- [x] Request lifecycle trace no longer ends in `no_selection_published` for the
  second outstanding request.
- [x] Docs record the new behavior and remaining concurrency limits.

## Root Cause

The failed `requests=2`, `concurrency=2` run was not a provider execution
failure and was not a request ID collision. The second worker received provider
ACK Data but never reached ACK decryption success. The provider attached a
wrapped hybrid message key on ACKs for the first worker, then reused the same
send key for ACKs to the second worker without attaching another wrapped key.
The second worker has a separate requester identity and process-local key cache,
so it cannot decrypt ACKs using the first worker's cached key.

Provider ACK and RESPONSE publications are therefore treated as request-scoped
receiver messages. The provider attaches a wrapped message key for each
ACK/RESPONSE and does not mark that key as globally wrapped for these messages.
That keeps concurrent workers independent while preserving the existing hybrid
envelope format.

## Current Limit

This feature verifies fairness for two concurrent full-network NativeTracer
requests on the default shared-backbone layout. Larger concurrency levels should
be validated as a small campaign before making broader latency or throughput
claims.

## Evidence Commands

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network \
  --core-trace \
  --assignment default \
  --role-execution-delay-ms 75 \
  --requests 2 \
  --concurrency 2 \
  --out /tmp/ndnsf-di-014-c2-provider-rebuilt \
  --provider-check-timeout 60
```

Observed result: `SUCCESS`, `successCount=2`, `failureCount=0`,
`meanMs=492.59`, `p95Ms=522.13`.
