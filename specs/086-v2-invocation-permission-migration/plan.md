# Implementation Plan: V2 Invocation And Permission Migration

## Context

The current provider callback parses V2 first and then falls back to V1. V1
encodes split service/function components and a provider Bloom filter in the
request name. The current encrypted PermissionResponse path already carries
permission kind and policy epoch, but `UserPermissionTable` discards both and
stores a deprecated token alongside split-name data.

## Constitution Check

- **Security first**: no authorization or token assertion is weakened; all
  current negative security regressions are mandatory gates.
- **Evidence before deletion**: every removed declaration, definition, caller,
  registration, build entry, and test has an exact inventory.
- **Core/application boundary**: only generic invocation and authorization
  mechanisms change; DI/Repo/UAV policy remains outside Core.
- **Network validation**: final normal and Targeted acceptance uses MiniNDN.
- **Incremental rollback**: implementation and evidence closure are separate,
  independently revertible commits.

## Architecture Decision

1. Replace `UserPermissionTable` with `ServiceAuthorizationTable`.
2. Key records by canonical `/<provider>/<serviceName...>` URI.
3. Store unified service name, `PermissionKind`, and policy epoch.
4. Reject a lower policy epoch for an existing record; allow same/newer epoch
   replacement. Empty snapshots replace all records for the role.
5. Keep PermissionEntry token decode only at the wire boundary and ignore it.
6. Remove V1 public APIs and wire fallback in one release. This Experimental
   branch has no stable external ABI promise; downstream in this monorepo is
   migrated and verified before deletion.
7. Retain the current V2 normal, Targeted, collaboration, NAC-ABE, bootstrap,
   and one-time token paths unchanged.

## Implementation Slices

### Slice A - Freeze evidence

Capture symbols, registrations, build entries, tests, ABI decision, and entry
baseline under `evidence/`.

### Slice B - Authorization table

Add `ServiceAuthorizationRecord` and `ServiceAuthorizationTable`; migrate
ServiceUser, ServiceProvider, and encrypted permission tests. Verify behavior
before deleting old representation.

### Slice C - Remove V1 invocation

Remove V1 user API and request helpers, remove the provider fallback and V1
decrypt callback chain, then remove BloomFilter source/build entries.

### Slice D - Remove legacy permission discovery

Remove NDNSD token-name parsing/decryption callbacks while retaining direct
controller PermissionResponse acquisition. Keep generic discovery only where
it has a non-token purpose.

### Slice E - Acceptance

Run source/build scans, CodeGraph audit, full tests, security regressions, and
matched MiniNDN normal/Targeted smoke. Record rollback and close parent gates.

## Files Expected To Change

- `ndn-service-framework/ServiceAuthorizationTable.hpp` (new)
- `ndn-service-framework/UserPermissionTable.hpp` (remove)
- `ndn-service-framework/ServiceUser.hpp/.cpp`
- `ndn-service-framework/ServiceProvider.hpp/.cpp`
- `ndn-service-framework/utils.hpp/.cpp`
- `ndn-service-framework/BloomFilter.hpp/.cpp` (remove)
- `tests/unit-tests/encrypted-permission-response.t.cpp`
- `tests/unit-tests/*v2*` or focused new unit tests
- `examples/wscript`, `tests/wscript`, root build metadata if referenced
- current Core docs and Spec 084 gate evidence

## Verification

See `contracts/test-matrix.md`. Security checks are mandatory because the
permission representation and request dispatch path are security-sensitive.

## Rollback

The implementation is one independently revertible child commit followed by a
separate evidence/closure commit. No later child is required to restore the
pre-086 tree.
