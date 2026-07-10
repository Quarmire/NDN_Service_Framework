# Implementation Plan: Core Boundary And Fail-Closed Execution Leases

**Branch**: `085-core-boundary-fail-closed-leases` | **Date**: 2026-07-10

## Summary

Add a generic provider-local execution lease state machine to Core C++ with a
thin Python binding, carry DI lease operations over an ordinary secured V2 DI service, and move
application policy out of the generic Python surface. No coordinator or global
refCount is required for correctness.

## Technical Context

- Python 3.8+; existing C++17 Core remains wire/security authority.
- Existing V2 `ServiceProvider`/`ServiceUser` Python bindings carry DI payloads.
- No new TLV or Core service name is introduced.
- MiniNDN is the final network validation path.
- Parent experiment and removal gates are mandatory.

## Constitution Check

- Canonical Dynamic Runtime: PASS; lease service uses unified V2 serviceName.
- Security In Data Path: PASS; normal/Targeted security remains mandatory.
- CodeGraph First: PASS; current callers and dirty overlap are recorded.
- Spec Driven: PASS; contracts/tasks precede edits.
- Right Verification Scope: PASS; unit, security, failure, and MiniNDN gates.
- Dirty Worktree Safety: PASS for T001; pre-existing Core, Repo, docs, and
  workflow edits were verified and committed in four ownership-scoped commits.
  T002-T003 remain mandatory before production edits.

## Target Architecture

```text
Core C++ ndn-service-framework/ExecutionLease.hpp/.cpp
  GenericExecutionLease + state/reason enums
  ProviderExecutionLeaseTable
  opaque resourceBindingProof bytes; no DI model/stage/refCount policy

Python binding/runtime_telemetry adapter
  thin conversions over the C++ state machine
  no second lease algorithm

DI ndnsf_distributed_inference/deployment.py
  DeploymentRecord (descriptive)
  Python LeaseOperationRequest/Response codec and client/adapter
  prepare all -> commit all -> provider validate+activate -> execute
  otherwise abort/replan; completion releases in finally path

DI C++ ExecutionLeaseService.hpp/.cpp
  same versioned payload codec
  /Inference/Control/Lease handler for di-native-provider
  owns one Core ProviderExecutionLeaseTable per provider process

NativeProviderHandler + Python user_driver.py
  user driver runs DistributedLeaseTransaction
  assignment carries provider lease binding
  handler validate+activates before model work and releases afterward

DI artifact_deployment.py
  ExecutionArtifact/Spec/Context and materialization

Repo py_repoclient
  RepoDataPlaneProducer adapter

Generic ndnsf
  invocation/security/collaboration/large-data/status/telemetry only
```

## Lease State Machine

```text
NONE --prepare--> PREPARED --commit--> COMMITTED --activate--> EXECUTING
                       |                    |                      |
                       +--abort--> ABORTED  +--abort--> ABORTED   +--release--> RELEASED
                       +--expire--> EXPIRED +--expire--> EXPIRED  +--hard-deadline--> EXPIRED
```

- Provider creates a random boot epoch at table construction.
- Idempotency key is bound to requester, request, service, plan digest, roles,
  and opaque resource binding.
- Repeating the same operation returns the prior result.
- Same key with different content returns `LEASE_IDEMPOTENCY_CONFLICT`.
- Unknown/stale transitions fail closed.
- Renewal is allowed for PREPARED, COMMITTED, or EXECUTING leases before their
  applicable deadline.
- The provider execution handler atomically validates COMMITTED state and
  transitions to EXECUTING before business logic, then releases in a finally
  path. EXECUTING has a separate bounded hard deadline so a crashed handler
  cannot pin resources forever.
- Provider-local eviction queries COMMITTED and EXECUTING bindings.

## Wire/Application Contract

Service name: `/Inference/Control/Lease`.

Network operations: `PREPARE`, `COMMIT`, `ABORT`, `RENEW`, `RELEASE`.
Payload encoding is versioned deterministic JSON owned by DI. Provider identity
comes from the selected provider and authenticated NDNSF path, not a trusted
payload field. Responses echo operation, lease ID, epoch, state, expiry, and
typed reason.

`VALIDATE_AND_ACTIVATE` is provider-local, not a network lease operation. The
actual DI execution handler calls it atomically using the authenticated request
context and canonical binding proof immediately before business logic.

Known providers use Targeted after normal Targeted token bootstrap. The service
does not bypass permissions, NAC-ABE, UserToken/ProviderToken, replay checks, or
provider authorization.

Fault model: authenticated users/providers may crash, restart, lose, delay, or
duplicate messages but are not Byzantine. Provider authority guarantees local
capacity safety under any input; user-side all-provider atomicity assumes the
authenticated transaction does not intentionally fabricate a global commit set.

## Migration Waves

1. Resolve dirty target ownership and freeze tests.
2. Add Core C++ generic lease types/table, Python bindings, and parity tests.
3. Add one DI C++/Python payload contract, native provider service, Python
   transaction/client, and cross-language fixtures.
4. Route existing deployment calls through DI transaction; remove local fallback
   and global refCount authority.
5. Move execution artifact/materializer API and callers to DI.
6. Move Repo producer import/adapter to `py_repoclient` without storage changes.
7. Remove migrated generic exports and run forbidden-symbol gates.
8. Run full regressions and matched MiniNDN acceptance.

## Exact Source Scope

Primary current files:

- `pythonWrapper/ndnsf/runtime_telemetry.py`
- new `ndn-service-framework/ExecutionLease.hpp`
- new `ndn-service-framework/ExecutionLease.cpp`
- `wscript`
- `pythonWrapper/src/ndnsf/_ndnsf.cpp`
- `pythonWrapper/ndnsf/service.py`
- `pythonWrapper/ndnsf/__init__.py`
- `NDNSF-DistributedInference/ndnsf_distributed_inference/runtime_v1.py`
- `NDNSF-DistributedInference/ndnsf_distributed_inference/artifact_deployment.py`
- `NDNSF-DistributedInference/ndnsf_distributed_inference/__init__.py`
- `NDNSF-DistributedInference/ndnsf_distributed_inference/merge_provider.py`
- new `NDNSF-DistributedInference/ndnsf_distributed_inference/deployment.py`
- new `NDNSF-DistributedInference/cpp/ndnsf-di/ExecutionLeaseService.hpp`
- new `NDNSF-DistributedInference/cpp/ndnsf-di/ExecutionLeaseService.cpp`
- `NDNSF-DistributedInference/cpp/ndnsf-di/NativeProviderHandler.hpp`
- `NDNSF-DistributedInference/cpp/ndnsf-di/NativeProviderHandler.cpp`
- `examples/DI_NativeProviderExecutable.cpp`
- `examples/python/NDNSF-DistributedInference/native_di_tracer/user_driver.py`
- `examples/wscript`
- `NDNSF-DistributedRepo/pythonWrapper/py_repoclient/`
- `Experiments/NDNSF_DI_NativeTracer_Minindn.py` for generated lease-service
  permissions and acceptance wiring

Tests are listed by exact path in `tasks.md`. Production changes outside this
list require a plan amendment and caller scan.

## Rollback

Each wave is a separate commit. Migration commits precede export/deletion
commits. Revert deletion first, then migration, then new types. No database or
wire migration is part of this feature.
