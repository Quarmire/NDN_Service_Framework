# Feature Specification: NDNSF Occam Second Pass

**Feature**: `094-ndnsf-occam-second-pass`  
**Created**: 2026-07-11  
**Status**: Draft

## Purpose

Remove mechanisms that have no remaining caller, no runtime effect, duplicate a
validated canonical path, or produce misleading evidence across NDNSF Core,
NDNSF-DI, NDNSF-UAV-APP, and NDNSF-DistributedRepo. Preserve mechanisms that
are required for security, persistence, distributed correctness, application
semantics, or a bounded migration contract.

## User Scenarios & Testing

### User Story 1 - One Trustworthy DI Execution Path (Priority: P1)

An NDNSF-DI developer uses the threaded open-loop driver for measured offered
load and cannot accidentally select the process-pool mode whose worker startup
delay invalidates the schedule. RPS experiments use the canonical NativeTracer
MiniNDN harness and its complete acceptance metrics.

**Independent Test**: CLI, runtime profile, GUI, campaign wrappers, tests, and
documentation expose `threaded` and `child` only; a 60-second threaded MiniNDN
smoke completes and records scheduling, dependency, success, and latency data.

### User Story 2 - One Current DI GUI And Artifact API (Priority: P1)

An operator uses the current USER/PROVIDER/CONTROLLER tabs and the version-2
profile without a second set of script-role tabs or an obsolete profile schema.
DI callers use `artifact_references` without a second `repo_manifests` keyword.

**Independent Test**: GUI headless and Tk tests load version-2 profiles and run
all three roles; old profile/API names are rejected rather than silently
selecting a second path.

### User Story 3 - Persistent Repo Has One Storage Model (Priority: P1)

A Repo developer cannot instantiate a production-facing memory-only Repo. The
authoritative store is SQLite and the bounded memory layer remains a cache.
Tests use temporary SQLite or a test-local fake instead of a public memory-only
factory.

**Independent Test**: C++ Repo builds and tests pass with no public
`InMemoryRepoStore`, `makeMemoryRepoStore`, or default memory-backed
`RepoCore`/`RepoNode` constructor; restart persistence remains verified.

### User Story 4 - Public Options Have Runtime Effect (Priority: P1)

Repo callers see no accepted-but-ignored `producer_retention_s` or
`isolated_runtime` option, and operation status contains the typed state once
without a redundant `legacyStatus` copy.

**Independent Test**: exact symbol scans are empty, Repo focused tests pass,
and MiniNDN Repo operations still complete over the canonical runtime.

### User Story 5 - Future Occam Audits Are Actionable (Priority: P2)

A maintainer can run the Occam audit and receive findings for prohibited
mechanisms rather than false positives for correctly owned app types, abstract
methods, optional handlers, or typed operation status.

**Independent Test**: audit fixtures distinguish active/test/docs/history and
the repository has zero active findings for the new prohibited-mechanism rules.

## Functional Requirements

- **FR-001**: Every removal MUST have an exact caller inventory and a named
  canonical replacement before code is changed.
- **FR-002**: DI open-loop measured load MUST expose `threaded` as the canonical
  mode; `child` MAY remain for process-isolation diagnostics and closed-loop
  compatibility; `process-pool` and its private worker-batch protocol MUST be
  removed.
- **FR-003**: The obsolete runtime-aware RPS sweep MUST be removed because it
  uses deterministic execution and classifies stability without scheduling,
  throughput, dependency, or malformed-trace gates. Documentation MUST point to
  the canonical harness recipe.
- **FR-004**: The old `RuntimeGuiProfile`, `RuntimeRoleProfile`, profile
  load/write helpers, and three duplicate Script Role tabs MUST be removed.
  Version-2 `ThreeRoleGuiProfile` and current direct USER/PROVIDER/CONTROLLER
  tabs MUST remain authoritative.
- **FR-005**: DI public inference/deployment APIs MUST accept
  `artifact_references` only. The unused `repo_manifests` alias and selection
  shim MUST be removed without changing artifact retrieval semantics.
- **FR-006**: Repo C++ public API MUST require an explicit persistent/tiered
  store. Public memory-only store classes/factories and default memory-backed
  constructors MUST be removed. Test-only fakes MAY exist inside test sources.
- **FR-007**: The Python Repo runtime MUST remove `producer_retention_s` and
  `isolated_runtime`, including CLI/harness forwarding, because both values are
  ignored.
- **FR-008**: Typed `ServiceOperationStatus` MUST remain authoritative;
  redundant `legacyStatus` metadata MUST be removed. Repo domain capability
  fields MUST remain inside `ProviderCapabilityHint.service_payload`, but the
  misleading local variable `legacy_fields` MUST be renamed.
- **FR-009**: The Occam audit MUST use path/ownership-aware rules and MUST add
  direct recurrence rules for the mechanisms removed by this feature.
- **FR-010**: The bounded mixed ACK reader MUST remain until its existing
  next-major-release or 2026-12-31 gate; this feature MUST NOT weaken security,
  token validation, permission encryption, fail-closed leases, or typed ACK
  writers.
- **FR-011**: Core stream contracts/C++ algorithms, UAV H264/FEC/ROI policy,
  Repo persistence/replication/catalog/repair, and DI planning/runtime/cache/
  long-context mechanisms MUST remain unless a separate caller-and-evidence
  gate proves redundancy.
- **FR-012**: Every source removal MUST have focused regression coverage and
  the final integrated verification MUST include Python tests, C++ builds/tests,
  and relevant MiniNDN Core/DI/Repo/UAV checks.
- **FR-013**: Proposal slides and papers MUST NOT be modified.

## Success Criteria

- **SC-001**: All removed public symbols/options have zero active callers and
  zero active Occam findings.
- **SC-002**: The current GUI/profile, threaded DI path, persistent Repo path,
  typed operation status, and artifact-reference path pass focused tests.
- **SC-003**: Full Python regression has no new failure and affected C++ Repo,
  Core, DI, and UAV targets build and pass their focused tests.
- **SC-004**: At least one 60-second MiniNDN DI run and relevant Repo/UAV/Core
  network regressions pass, or any environmental block is recorded as missing
  evidence rather than a pass.
- **SC-005**: The final report lists every candidate as REMOVE, CONSOLIDATE,
  KEEP, or DEFER with rationale, owner, and gate for deferred items.
- **SC-006**: Net maintained source decreases; generated files, historical
  specs, and evidence do not count as runtime simplification.

## Non-Goals

- Removing capabilities merely because their implementation is large.
- Replacing SQLite, NDN SegmentFetcher, stream semantics, NAC-ABE, leases, or
  typed envelopes with simpler but incorrect behavior.
- Breaking the bounded mixed ACK migration contract early.
- Refactoring large translation units without deleting a duplicate mechanism.
- Modifying proposal materials.

## Assumptions

- Source compatibility for unused experimental aliases is not a product
  requirement when exact caller inventory is empty.
- Historical specs and tests may mention removed names as negative fixtures.
- MiniNDN remains the final network validation environment.
