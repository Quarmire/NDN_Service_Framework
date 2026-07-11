# Traceability

| Requirement / criterion | Tasks | Verification |
|---|---|---|
| FR-001, SC-005 | T001-T004, T025 | Caller inventory, disposition matrix, audit |
| FR-002 | T005-T006, T008 | Parser/profile/GUI tests and symbol scan |
| FR-003 | T007-T008 | Removed sweep scan and canonical recipe check |
| FR-004 | T009, T011 | Tk/headless GUI tests and profile round trip |
| FR-005 | T010-T011 | DI API tests and old-keyword negative scan |
| FR-006 | T012-T013, T017 | Repo C++ build, cache/restart tests, symbol scan |
| FR-007 | T014-T015, T017 | Repo focused tests and exact option scan |
| FR-008 | T016-T017 | Typed status/capability tests |
| FR-009, SC-001 | T018-T020 | Occam audit fixture tests and fail-on-active |
| FR-010, FR-011 | T002-T004, T021-T023 | Adversarial audit and integrated regressions |
| FR-012, SC-002-SC-004 | T008, T011, T017, T021-T023 | Focused/full/build/MiniNDN evidence |
| FR-013 | T024 | Proposal-path diff scan |
| SC-006 | T001, T024 | Before/after maintained-source report |
| Convergence: persistent callers | T028-T029 | Full C++ build and Core HELLO MiniNDN |
| Convergence: Repo locator path | T030-T031 | Repo focused tests and Repo MiniNDN |

Every task is mapped to at least one user story in `tasks.md`. No task may be
closed from a checked box alone; its verification path must exist in evidence.
