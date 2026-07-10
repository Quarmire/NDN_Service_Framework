# Implementation Plan: Targeted Parallel Repo Control Plane

## Architecture

1. Extend `NativeServiceUser` and `ServiceUser` with
   `request_service_targeted` and `request_service_targeted_async`.
2. Register Python handlers through C++ `NormalAndTargeted`, reusing the same
   authenticated request-context adapter for both modes.
3. Make Targeted token batch size configurable while preserving one-time token
   consumption and automatic bootstrap/refill.
4. Add a Repo replica-call coordinator. One dispatcher thread submits all
   Targeted async calls; callbacks collect responses by Repo; the caller waits
   under one total deadline.
5. Parallelize capacity reserve/release and replicated write receipt collection.
   Keep Normal request/ACK/Selection as bounded fallback for older providers.
6. Add control timing/counter output and rerun matched MiniNDN campaigns.

## Ownership Boundary

- NDNSF core owns Targeted Python bindings, provider invocation mode, token
  batches, and generic sync/async semantics.
- NDNSF-REPO owns replica fan-out, receipt validation, fallback policy, and Repo
  metrics.

## Safety

Targeted requests are submitted only after provider selection or explicit
configuration. Retries reuse operation IDs. A timeout never fabricates a
receipt. The normal path remains available but is not raced concurrently with
Targeted unless the Targeted attempt has terminated.

## Validation

- C++ build and existing dynamic-runtime security tests.
- Python wrapper and Repo unit tests.
- MiniNDN Targeted bootstrap/fast-path smoke.
- Matched 60-second read-heavy c16/2-RPS and write-heavy c4/0.5-RPS campaigns.
