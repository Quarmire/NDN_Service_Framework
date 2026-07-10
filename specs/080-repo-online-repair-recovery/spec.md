# Feature Specification: Online Replica Repair After Recovery

**Feature Branch**: `Experimental`

**Created**: 2026-07-10

**Status**: Complete

## Goal

Restore objects committed with RF=3/W=QUORUM during a one-node outage back to
three durable replicas after that node restarts, without interrupting writes,
fabricating receipts, or requiring an operator to copy objects manually.

## User Stories

### US1 - Continue During Failure (P1)

A writer can continue committing RF=3/W=QUORUM objects to two healthy Repo
nodes while one desired node is offline.

### US2 - Rejoin And Discover Missing Objects (P1)

After the failed Repo restarts, its catalog sidecar rejoins membership, merges
peer catalog deltas, and discovers objects written while it was unavailable.

### US3 - Durable Online Repair (P1)

A surviving or recovered sidecar creates and claims durable repair jobs,
copies exact packet-backed or opaque objects through the existing NDNSF Repo
service path, and marks jobs complete only after the target confirms storage.

### US4 - Measured Recovery Evidence (P1)

A MiniNDN campaign correlates post-failure writes with repair events on the
recovered node and reports repair coverage and recovery latency.

## Functional Requirements

- **FR-001**: RF=3/W=QUORUM writes MUST remain available with two validated
  durable receipts while one Repo is offline.
- **FR-002**: A degraded committed manifest MUST preserve desired RF=3 while
  listing only confirmed receipt owners.
- **FR-003**: Restarting a failed Repo in the MiniNDN campaign MUST also restart
  its catalog/auto-repair sidecar with the same identity, storage directory,
  controller, group, and peers.
- **FR-004**: The recovered sidecar MUST publish fresh membership and merge
  catalog deltas before it can be selected as a repair target.
- **FR-005**: Catalog planning MUST identify a fresh persistent recovered Repo
  that does not hold an RF=3 object as an under-replication target.
- **FR-006**: Repair jobs MUST remain durable, leased, idempotent, retryable,
  and uniquely identified by object, generation, source, and target.
- **FR-007**: Object bytes MUST move through the existing validated
  `CATALOG_REPAIR` / `STORE_PACKET_PULL` service path; the harness MUST NOT copy
  SQLite files or payloads directly.
- **FR-008**: A repair job MUST complete only after target storage succeeds and
  source hash, manifest hash, packet names, and packet wires validate.
- **FR-009**: Campaign lifecycle rows MUST include `objectName` so writes can be
  correlated with repair events.
- **FR-010**: Campaign output MUST report outage-window successful writes,
  recovered-target repair events, repair coverage, first/last repair latency,
  and unrepaired object names.
- **FR-011**: Measurement counters MUST exclude seed/readiness traffic.
- **FR-012**: Existing Specs 073-079 regressions MUST continue to pass.
- **FR-013**: No proposal slides, NDN-SVS timing, or NDNSF security bypasses may
  be introduced.

## Success Criteria

- **SC-001**: Unit tests prove a fresh recovered node is selected as the one
  missing target for a desired RF=3 object held by two live replicas.
- **SC-002**: Unit tests prove stale, non-persistent, or already-owning nodes
  are not selected as repair targets.
- **SC-003**: A 60-second MiniNDN campaign commits at least one write during
  RepoA downtime with exactly two validated receipts.
- **SC-004**: After RepoA restarts, at least one object written during the
  outage is durably repaired to RepoA through the NDNSF repair path.
- **SC-005**: The campaign reports machine-readable repair coverage and
  latency, and no successful write reports fewer than W receipts.
- **SC-006**: Focused Python/C++/Targeted/NAC regressions, Spec Kit consistency,
  CodeGraph impact review, and GSD acceptance pass.

## Non-Goals

- No consensus protocol, distributed transaction, or immediate synchronous
  restoration before write success is returned.
- No direct database replication or out-of-band file copy.
- No guarantee that every missing replica repairs within a fixed short window
  under arbitrary partitions; incomplete repair remains explicit evidence.
- No proposal-slide changes.
