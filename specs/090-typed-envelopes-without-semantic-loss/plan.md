# Implementation Plan: Typed Envelopes Without Semantic Loss

## Constitution Check

- Canonical dynamic runtime and V2 service naming are unchanged.
- Existing security, NAC-ABE, token, replay, and permission paths remain.
- Spec Kit and CodeGraph precede implementation.
- Full module, security, persistence, exact-wire, and MiniNDN validation match
  the cross-project risk.

## Architecture

1. Add Core Python `AckCompatibilityMode`, `AckCompatibilityCounters`, and
   `decode_provider_capability_ack()` around the existing ACK field codec.
2. Version `ProviderCapabilityHint` as
   `ndnsf-provider-capability-v2`; accept v1 typed envelopes during the bounded
   reader epoch, reject unknown versions.
3. Compare a frozen alias map only in mixed mode. Typed values are
   authoritative and conflicts are observable.
4. Switch DI Python, native DI C++, and Repo Python ACK producers to one
   `providerCapabilityHint` field. Keep app details in versioned
   `servicePayload`.
5. Migrate consumers and experiment diagnostics to the shared decoder.
6. Keep stored schemas unchanged because ACK aliases are transient wire
   metadata, not authoritative persisted records.

## Compatibility Epoch

- Start: child 090 implementation commit.
- Current producer mode: typed-only.
- Reader default: typed-only.
- Explicit mixed reader: `NDNSF_ACK_COMPATIBILITY_MODE=mixed` or direct API.
- Legacy-only reader support expires at the next major release or 2026-12-31,
  whichever is earlier.
- Removal exit: current producer scan is zero, mixed campaign has zero
  unexplained conflicts, and legacy-use counters remain zero in typed-only
  acceptance.

## Security And Persistence

ACK payload remains inside the existing signed/encrypted NDNSF path. Unknown or
malformed typed envelopes fail closed. No database, exact Data wire, plan,
cache, mission, or config migration is required.

## Rollback

Implementation and closure are separate commits. Reverting implementation
restores dual producer emission and old readers without changing stored state.

## Touch Points

- Core codec/API: `pythonWrapper/ndnsf/runtime_telemetry.py` and
  `pythonWrapper/ndnsf/__init__.py`.
- DI producers/consumers: `NDNSF-DistributedInference/ndnsf_distributed_inference/provider.py`,
  `runtime_v1.py`, `deployment.py`, native
  `cpp/ndnsf-di/NativeProviderReadiness.cpp`, NativeTracer user/harness.
- Repo producer/consumers:
  `NDNSF-DistributedRepo/pythonWrapper/py_repoclient/orchestration.py` and
  `__init__.py`.
- Contracts/tests: `tests/python/test_ndnsf_ack_compatibility_v2.py`, DI/Repo
  envelope tests, producer scans, persistence/exact-wire tests, and MiniNDN
  harness summaries.
