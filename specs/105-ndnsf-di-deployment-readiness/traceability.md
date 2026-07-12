# Traceability Matrix

## Requirements to Design, Tasks and Evidence

| Requirement | Design/contract | Tasks | Closing evidence |
|---|---|---|---|
| FR-001 | `ExecutionEvidence`; execution-evidence contract | T009, T013-T014, T018, T021-T023 | evidence gate results |
| FR-002 | release-gate contract | T010, T019-T020, T024-T026, T030 | evidence gate results |
| FR-003 | migration sequence | T001-T002, T027-T029 | historical evidence correction |
| FR-004 | canonical pilot path | T005-T006, T033, T037-T044 | Qwen correctness |
| FR-005 | experiment matched baseline | T033, T038-T039, T045-T048 | correctness/performance reports |
| FR-006 | typed tensor contract | T031-T036 | focused C++ tests and Qwen correctness |
| FR-007 | `KvStateBinding` | T033, T040-T042 | cache correctness/fault cells |
| FR-008 | fixed workload boundary | T005, T033, T039, T044 | admission negative results |
| FR-009 | capability/telemetry split | T011, T049-T055 | telemetry validation |
| FR-010 | telemetry snapshot contract | T049, T052-T056 | telemetry validation/metrics |
| FR-011 | freshness fail-closed | T050-T051, T056-T061 | stale/configured-only cells |
| FR-012 | `PlanLease` predicates | T050, T057-T059 | plan decision matrix |
| FR-013 | bounded scheduler | T063, T066-T069, T077 | 1,000-wait stress |
| FR-014 | `ExecutionAttempt` | T012, T064, T070-T075 | attempt/fault evidence |
| FR-015 | one replacement | T065, T071-T073, T076-T078 | provider-loss matrix |
| FR-016 | terminal reasons contract | T012, T065, T069-T076 | negative/fault results |
| FR-017 | boot invalidation | T064, T074, T076, T078 | restart/cache-loss cell |
| FR-018 | security design | T006, T046-T048, T092, T095, T097 | MiniNDN application-security-path evidence and explicit physical deferral |
| FR-019 | operator CLI/systemd | T079-T091 | staging and second-operator evidence |
| FR-020 | release/artifact binding | T080-T089, T097 | release/security audits |
| FR-021 | migration/rollback | T081, T088-T093 | local staged upgrade/rollback drill |
| FR-022 | observability design | T075, T085, T092-T095 | metrics, local soak, integrated regression |
| FR-023 | experiment plan | T006, T030, T045-T048, T061-T062, T077-T078, T092-T094 | all local acceptance records |
| FR-024 | MiniNDN-only scope and Spec 106 handoff | T006, T090, T092-T094, T098 | candidate gate and physical deferral |

## Success Criteria to Tasks

| Criterion | Tasks | Verdict source |
|---|---|---|
| SC-001 | T018-T030 | evidence gate results |
| SC-002 | T033-T048 | Qwen MiniNDN performance |
| SC-003 | T045-T048 | matched latency/decomposition report |
| SC-004 | T049-T062 | telemetry/plan validation |
| SC-005 | T063-T069, T077 | bounded scheduler report |
| SC-006 | T064-T078 | fault recovery report |
| SC-007 | T079-T092 | two clean local staging runs |
| SC-008 | T092-T094 | 24-hour local MiniNDN soak |
| SC-009 | T088-T093 | local staged upgrade/rollback drill |
| SC-010 | T026, T098-T100 | final release gate and audits |

## User Story Independence

- **US1** closes evidence truth without requiring compute/runtime changes.
- **US2** closes real bounded Qwen correctness/performance after US1 truth.
- **US3** closes measured placement on the accepted real runtime.
- **US4** closes bounded scheduling/recovery on the accepted plan semantics.
- **US5** packages and locally operates only capabilities already accepted by
  US1-US4; Spec 106 owns physical acceptance.

No task exists solely to increase abstraction or compatibility. Every production
mechanism maps to a requirement, failure mode and closing artifact.
