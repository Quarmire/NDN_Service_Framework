# Implementation Plan: NDNSF-DI Advisory Coordinator

**Branch**: `048-di-advisory-coordinator` | **Date**: 2026-07-05 |
**Spec**: [spec.md](spec.md)

## Summary

Add a lightweight advisory coordinator for NDNSF-DI multi-user planning while
preserving the Spec047 user-side planner. The coordinator receives plan intents,
uses the same runtime-aware provider scoring as the user, applies a fairness
penalty to avoid overusing providers across a short intent window, and returns
non-binding suggestions. The user only accepts a suggestion after checking
freshness, proof, request/template match, current candidate validity, and
provider lease validity. Generic intent/suggestion freshness, proof, nonce, and
opaque payload fields come from SPEC049's NDNSF core coordination envelope;
NDNSF-DI only interprets the DI payload.

## Technical Context

**Language**: Python 3 runtime contracts first; C++ integration is future work.

**Primary Files**:

- `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- `pythonWrapper/ndnsf/coordination.py`
- `NDNSF-DistributedInference/ndnsf_distributed_inference/__init__.py`
- `tests/python/test_ndnsf_di_advisory_coordinator.py`
- `specs/049-core-coordination-envelope/quickstart.md`

**Dependencies**: Existing Spec047 runtime metadata, SPEC049 coordination
envelope, `PlanTemplate`, `RuntimeAssignment`, `GenericAckMetadata`,
`GenericAdmissionLease`, `ProviderNetworkMatrix`, and
`score_runtime_candidate`.

**Constraints**:

- Suggestions never authorize execution.
- Provider leases remain the resource authority.
- The coordinator is disabled by default.
- Proof is deterministic MVP proof for regression coverage; full NDNSF
  certificate-signed proof can be added later.

## Design

1. `PlanIntent` captures a user's planning request.
2. `AdvisoryCoordinator` opens a short logical window over valid intents.
3. For each intent, the coordinator enumerates valid role/provider combinations
   using existing runtime-aware scoring.
4. It adds a provider-use fairness penalty across previous suggestions in the
   same window.
5. It returns `AdvisorySuggestion` with assignment, expiry, score details, and
   proof.
6. `merge_advisory_suggestion` validates the suggestion and then rebuilds the
   assignment from the user's current candidate set.

## Validation

Run:

```bash
PYTHONPATH=NDNSF-DistributedInference python3 tests/python/test_ndnsf_di_advisory_coordinator.py
PYTHONPATH=NDNSF-DistributedInference python3 tests/python/test_ndnsf_di_runtime_aware_planner.py
PYTHONPATH=NDNSF-DistributedInference python3 tests/python/test_ndnsf_di_runtime_v1.py
```

MiniNDN is not required for this MVP because the feature is a transport-neutral
planner contract. Later C++/wire integration should add a MiniNDN coordinator
service test.

## Constitution Check

- **Canonical Dynamic Runtime**: PASS. No generated stubs or service/function
  split added.
- **Security Is Part Of The Data Path**: PASS. Suggestions cannot bypass
  provider permissions, NAC-ABE, tokens, or leases.
- **CodeGraph First, Source Verified**: PASS. Runtime symbols were explored via
  CodeGraph before edits.
- **Spec-Driven Changes For Durable Work**: PASS. SPEC048 records design and
  tasks.
- **Verify With The Right Scope**: PASS. Unit tests cover the transport-neutral
  MVP; MiniNDN is deferred until network integration exists.
