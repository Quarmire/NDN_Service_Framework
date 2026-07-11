# Feature Specification: V2 Invocation And Permission Migration

**Feature Branch**: `Experimental`
**Created**: 2026-07-10
**Status**: In progress
**Parent**: `specs/084-ndnsf-occam-simplification`

## Purpose

Remove the legacy V1 split-name/Bloom-filter invocation path while preserving
the provider/service authorization state and security properties used by the
current V2 protocol.

## User Stories

### User Story 1 - One canonical invocation protocol (Priority: P1)

As an NDNSF application author, I use one unified `serviceName` request API and
one V2 wire-name grammar so that request behavior does not depend on a hidden
legacy fallback.

**Acceptance**: normal, Targeted, and collaboration calls use only V2 names;
the legacy `PublishRequest`, split `ServiceName/FunctionName`, Bloom-filter
request naming/parsing, and legacy provider request branch are absent.

### User Story 2 - V2 authorization survives cleanup (Priority: P1)

As a user or provider, I still receive controller-signed, target-certificate
encrypted permissions and can authorize a provider/service using the response
kind and policy epoch.

**Acceptance**: authorization records contain `providerServiceName`,
`serviceName`, `permissionKind`, and `policyEpoch`; no invocation token is
stored in this table.

### User Story 3 - Security behavior does not regress (Priority: P1)

As an operator, I retain NAC-ABE routing, one-time UserToken/ProviderToken,
replay rejection, certificate bootstrap, normal invocation, Targeted bootstrap,
and collaboration behavior after V1 removal.

## Functional Requirements

- **FR-001**: The only public request protocol MUST use unified `serviceName`
  V2 names.
- **FR-002**: The Core MUST provide a thread-safe `ServiceAuthorizationTable`
  keyed by canonical provider/service name.
- **FR-003**: Each authorization record MUST retain canonical provider/service
  name, unified service name, permission kind, and policy epoch.
- **FR-004**: PermissionResponse application MUST reject the wrong target or
  wrong permission kind and MUST install the response policy epoch.
- **FR-005**: The old PermissionEntry token wire field MAY be decoded for wire
  compatibility but MUST NOT be indexed or used for invocation authorization.
- **FR-006**: V1 `PublishRequest`, `parseRequestName`, V1 request-name builders,
  BloomFilter source/build entries, and the provider V1 fallback MUST be
  removed only after exact caller/registration proof.
- **FR-007**: Legacy NDNSD token-name permission installation and its decrypt
  callbacks MUST be removed; direct encrypted PermissionResponse remains the
  permission authority.
- **FR-008**: One-time UserToken/ProviderToken generation, echo checks, replay
  protection, Targeted token batches, and NAC-ABE policy routing MUST remain.
- **FR-009**: No Direct API/terminology alias may remain for Targeted calls.
- **FR-010**: Builds, tests, examples, and current docs MUST contain no active
  V1 or Bloom-filter invocation references; archived specifications are exempt.
- **FR-011**: Malformed and legacy V1 request names MUST fail closed without
  invoking a handler.
- **FR-012**: The migration MUST be independently revertible from later Occam
  children and MUST NOT modify proposal slides, NDN-SVS, or unrelated apps.

## Non-Functional Requirements

- **NFR-001**: Full C++ and focused Python tests pass.
- **NFR-002**: Security regressions pass without weakened assertions.
- **NFR-003**: Matched MiniNDN normal and Targeted workflows retain 100% smoke
  completion; p95 may not regress by more than 15% from the frozen baseline.
- **NFR-004**: Authorization-table operations are thread-safe and deterministic.
- **NFR-005**: Zero forbidden production symbols/build entries remain.

## Out Of Scope

- Removing the PermissionEntry token TLV from the wire schema.
- Changing NAC-ABE attributes, ServiceController certificate bootstrap, or
  one-time invocation token semantics.
- DI/Repo/UAV policy refactors owned by Specs 087-090.
- Proposal-defense slides.

## Success Criteria

- **SC-001**: Forbidden-symbol scan reports zero production matches for V1
  request helpers, `PublishRequest`, and BloomFilter.
- **SC-002**: Authorization-table unit tests cover user/provider records,
  replacement by policy epoch, wrong-kind rejection, and concurrent reads.
- **SC-003**: Full C++ suite, focused Python suite, and security aggregate pass.
- **SC-004**: Normal and Targeted MiniNDN smoke meet NFR-003.
- **SC-005**: CodeGraph reports no live V1 callback registration or caller.
