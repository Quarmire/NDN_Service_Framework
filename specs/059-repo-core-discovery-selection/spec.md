# Feature Specification: Repo Core Discovery Selection

**Feature Branch**: `059-repo-core-discovery-selection`

**Created**: 2026-07-08

**Status**: Draft

## User Story

As a DistributedRepo client, I need ACK selection to respect core
`ProviderCapabilityHint` readiness and drain state, so a provider that is
draining or unavailable is not selected merely because it still reports storage
capacity fields.

## Requirements

- **FR-001**: Repo ACK parsing MUST expose a reusable core discovery record when
  a typed `ProviderCapabilityHint` is present.
- **FR-002**: Repo capacity selection MUST skip providers whose core discovery
  record is not ready for new requests.
- **FR-003**: Legacy ACKs without typed core hints MUST remain usable and be
  treated as ready, preserving existing behavior.
- **FR-004**: If all ACK candidates are unready or draining, selector output MUST
  be empty instead of silently selecting an unavailable provider.
- **FR-005**: Tests MUST cover typed ready, typed draining, typed unready, and
  legacy-only fallback ACKs.

## Non-Goals

- Do not move Repo storage, catalog, or replica placement policy into NDNSF
  core.
- Do not introduce a persistent provider state cache.
- Do not replace the existing ACK metadata wire format.

## Success Criteria

- Repo py client tests prove core hint readiness is honored.
- Existing Repo/UAV/DI app core migration tests still pass.
- Core discovery tests still pass.

