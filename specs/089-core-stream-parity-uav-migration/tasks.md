# Tasks: Core Stream Parity And UAV Migration

## Phase 1 - Freeze And Audit

- [x] T001 [US1] Inventory C++/Python/UAV stream symbols, callers and parity gaps.
- [x] T002 [US1] [US2] Freeze session/reorder/duplicate/gap/skip/deadline/stale,
  pending count/bytes, overflow, metrics, malformed, unknown and adaptive vectors.
- [x] T003 Specify C++ ownership/threading and TLV/Python conversion; run strict
  structure and pre-implementation audit.

## Phase 2 - One Engine

- [x] T004 [US1] Add missing native pending/overflow observability test-first.
- [x] T005 [US1] Bind C++ stream value/state classes in `_ndnsf`.
- [x] T006 [US1] Add binding conversion and malformed/unknown tests.
- [x] T007 [US1] Replace Python reorder/producer/adaptive algorithms with thin
  native adapters.
- [x] T008 [US1] Run shared parity vectors and delete duplicate Python logic.
- [x] T009 [US2] Migrate UAV generic stream state to Core without moving policy.
- [x] T010 [US2] Run UAV FEC/video/authority/mission regressions.
- [x] T011 [US3] Add forbidden-use tests for static/finite objects.

## Phase 3 - Acceptance

- [ ] T012 Run full C++/Python/Core security regressions.
- [ ] T013 Run three matched UAV MiniNDN loss campaigns.
- [ ] T014 Verify rollback, post-audit/analyze/converge, parent T045-T051 and
  separate implementation/closure commits.
