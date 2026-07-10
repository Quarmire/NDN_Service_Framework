# Feature Specification: Repo Catalog Merge Large-Data Path

**Created**: 2026-07-10

**Status**: In Progress

## Goal

Replace large catalog delta merge request batches with one authenticated control
request plus exact-name segmented retrieval, while retaining inline merge for
small deltas and a compatibility fallback.

## Requirements

- **FR-001**: Catalog merge payloads at or below 6,000 encoded bytes MUST use
  the existing inline `CATALOG_MERGE` operation.
- **FR-002**: Larger payloads MUST be published as signed segmented Data and
  referenced by one `CATALOG_MERGE_PULL` request.
- **FR-003**: The pull request MUST carry exact source name, SHA-256, byte size,
  entry count, and schema version.
- **FR-004**: The target MUST reject unsupported schema, oversized payload,
  length mismatch, hash mismatch, malformed JSON, and entry-count mismatch.
- **FR-005**: Valid payloads MUST reuse `_merge_catalog_entries` and membership
  status handling without changing catalog semantics.
- **FR-006**: The producer MUST stop after success or failure.
- **FR-007**: Pull failure MUST fall back to bounded inline batches and be
  visible in logs.
- **FR-008**: Merge logs and campaign summaries MUST distinguish inline, pull,
  and fallback modes and record payload bytes, segments, batches, and duration.
- **FR-009**: Existing repair, finalization, Targeted security, and exact-data
  behavior MUST remain unchanged.
- **FR-010**: Proposal slides and NDN-SVS MUST NOT be modified.

## Success Criteria

- **SC-001**: Unit tests cover inline, pull, fallback, cleanup, and validation
  failures.
- **SC-002**: Repo Python/C++/Targeted/security/worker regressions pass.
- **SC-003**: One matched 60-second MiniNDN campaign remains 30/30, W=2, zero
  invalid repairs, and 4/4 strict outage repair.
- **SC-004**: The initial recovered-sidecar peer deltas use one pull each rather
  than 16 inline batches each.
- **SC-005**: Results honestly report merge and request latency, positive or
  negative, from the single planned run.

## Non-Goals

- No replacement for exact-name object transfer or generic stream semantics.
- No consensus or new trust root.
- No production SLO claim from one campaign.
