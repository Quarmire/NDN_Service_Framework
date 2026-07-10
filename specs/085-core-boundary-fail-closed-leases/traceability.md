# Traceability

| Requirement | Tasks | Evidence |
|---|---|---|
| FR-001 generic lease envelope | T005-T006, T010-T014 | Core/binding parity |
| FR-002 provider table operations | T005-T006, T010-T015 | state-machine tests |
| FR-003 typed rejection | T005-T008, T011, T018-T020 | failure matrix |
| FR-004 restart epoch | T008, T011, T020, T023 | restart tests/campaign |
| FR-005 all-provider commit | T007, T016, T019, T023 | transaction tests |
| FR-017 atomic execution activation | T005, T011, T019-T023 | activation/deadline tests |
| FR-006 partial failure cleanup | T007-T008, T019-T023 | failure/stress evidence |
| FR-007 remove broken fallback | T003, T026, T030-T032 | fallback regression |
| FR-008 descriptive deployment | T016, T021, T025-T026 | deployment tests |
| FR-009 provider-local eviction | T008, T021, T023 | pin/expiry tests |
| FR-010 move DI policy | T009, T024-T026, T029-T033 | import/caller evidence |
| FR-011 move Repo producer | T009, T027, T029-T033 | Repo tests/import evidence |
| FR-012 defer coordination | T009, T029-T030 | export scan |
| FR-013 security/invocation stable | T017-T018, T032 | security suite |
| FR-014 existing V2 wire path | T016-T20, T032, T035 | service/MiniNDN evidence |
| FR-015 dirty entry gate | T001, T038 | ownership record |
| FR-016 rollback/removal gates | T004, T030-T031, T038 | READY gates |
| FR-018 native/Python real-path parity | T007, T016-T23, T033-T036 | codec and native MiniNDN logs |
| FR-019 provider conflict keys | T005, T010-T11, T018, T021, T023 | overlap and concurrency evidence |

Supporting tasks: T002 protects FR-013/SC-005 baseline fidelity; T012-T013 are
part of FR-001/FR-002 canonical Core and binding implementation; T022 completes
FR-005/FR-010 DI API wiring; T028 implements FR-010 retry ownership.

| Success criterion | Tasks |
|---|---|
| SC-001 fail closed | T003, T026, T032, T035-T036 |
| SC-002 atomic multi-provider | T007, T019, T023, T035-T036 |
| SC-003 failure matrix | T005-T008, T020-T023 |
| SC-004 clean Core exports | T009, T024-T030 |
| SC-005 regressions | T015, T032-T033 |
| SC-006 MiniNDN thresholds | T034-T036 |
| SC-007 evidence/rollback | T001-T004, T030-T031, T037-T038 |
