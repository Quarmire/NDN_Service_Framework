# Feature Specification: Core Coordination Envelope

**Feature Branch**: `049-core-coordination-envelope`
**Created**: 2026-07-05
**Status**: Implemented MVP + Wire-Path Smoke
**Input**: The DI advisory coordinator should not own generic coordination
freshness, proof, nonce, and suggestion-envelope semantics that other NDNSF
applications may reuse.

## User Scenarios

### Scenario 1: An application sends a service-neutral coordination intent

An NDNSF application creates a coordination intent with request id, requester
name, service name, nonce, lifetime, payload schema, and opaque payload. The
application-specific payload may describe DI planning, UAV workflow hints, or
another future service workflow.

### Scenario 2: A coordinator returns a service-neutral suggestion

A coordinator returns a suggestion with suggestion id, intent id, request id,
service name, coordinator name, window id, expiry, proof, payload schema, and
opaque payload. NDNSF core can validate freshness and proof without knowing the
payload semantics.

### Scenario 3: NDNSF-DI uses the generic envelope

NDNSF-DI keeps model stages, fragments, role assignments, cache locality, and
provider scoring in the DI layer while reusing the core envelope for common
coordination fields.

### Scenario 4: A coordinator suggestion travels on the NDNSF service path

A coordinator registers a normal NDNSF service. A user sends a coordination
request through the existing Request/ACK/Selection/Response path or targeted
service path and receives coordination suggestions as ordinary NDNSF response
payload.

## Requirements

- **REQ-049-001**: Provide a generic coordination intent envelope outside the
  NDNSF-DI package.
- **REQ-049-002**: Provide a generic coordination suggestion envelope outside
  the NDNSF-DI package.
- **REQ-049-003**: Provide shared freshness, stable digest, and deterministic
  proof helpers for the MVP.
- **REQ-049-004**: Keep DI-specific fields in NDNSF-DI payload wrappers.
- **REQ-049-005**: Existing DI advisory API names must continue to work.
- **REQ-049-006**: Tests must prove core coordination behavior independently
  from DI and prove DI advisory behavior still passes.
- **REQ-049-007**: Provide a Python service transport wrapper that carries
  coordination requests and responses through the existing NDNSF service API.
- **REQ-049-008**: NativeTracer MiniNDN harnesses must be able to run pure
  user-side planning and advisory-coordinator planning as comparable modes.

## Non-Goals

- No new C++ wire protocol in this MVP.
- No replacement of provider admission leases, UserToken/ProviderToken,
  NAC-ABE, or permission checks.

## Success Criteria

- Core coordination tests pass without importing NDNSF-DI.
- DI advisory tests pass while importing the core coordination envelope.
- Existing runtime-aware planner and runtime-v1 tests continue to pass.
- Dry-run harness output shows a coordinator service, user driver coordination
  service argument, and pure/advisory RPS comparison commands.
