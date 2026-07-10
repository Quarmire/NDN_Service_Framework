# Requirements And Audit Traceability

## Audit Finding To Requirement

| Audit finding | Requirement/contract | Program tasks |
|---|---|---|
| Broken local lease import and undefined replacement authority | FR-003, FR-004; `di-lease-authority.md` | T003, T013-T018 |
| Repo implementation preselected while wire choice remained open | FR-010, FR-011; `repo-decision-gate.md` | T035-T044 |
| Current V2 authorization still uses legacy-shaped table/callback graph | FR-002, FR-008; `permission-v2-migration.md` | T019-T026 |
| Tasks used missing/ambiguous paths and wildcard targets | FR-020, FR-023 | T002, T024, all child audits |
| Performance thresholds could be chosen after results | FR-021; `experiment-gates.md` | T004-T009, T032, T044, T051 |
| One 128-task cross-project feature lacked rollback isolation | FR-022, FR-023; `child-feature-map.md` | T013, T019, T027, T035, T045, T052 |
| Repo private operations lacked an enforceable security boundary | FR-002, FR-011 | T037-T042 |
| Generic status and domain status could be incorrectly conflated | FR-016 | T052-T058 |
| C++/Python stream behavior and binding semantics were unspecified | FR-013, FR-014 | T045-T051 |
| Regression matrix and SC mapping were descriptive, not executable | FR-020, FR-021 | T008-T012, T061-T067 |

## Functional Requirements

| Requirement | Primary program tasks | Child verification/evidence |
|---|---|---|
| FR-001 ownership matrix | T002, T010, T061, T068 | final boundary graph and removal gates |
| FR-002 security invariants | T004, T022, T039, T062 | child 086/088 and final Core security evidence |
| FR-003 distributed leases/atomicity | T003, T013-T018 | child 085 concurrency/restart MiniNDN |
| FR-004 remove broken/local fallback | T003, T014, T018 | authority-loss regression and zero synthetic leases |
| FR-005 no application policy in Core | T016-T018, T029 | Core public-surface scan and child 085 acceptance |
| FR-006 DI-owned lifecycle | T015-T018, T027-T034 | child 085/087 deployment tests |
| FR-007 optional advisory coordination | T028-T034 | coordinator-off acceptance and retention experiment |
| FR-008 safe V1 removal | T019-T026 | child 086 symbol gates and security regressions |
| FR-009 Targeted terminology | T020, T022, T026 | V2 compile/API and matched Targeted evidence |
| FR-010 one selected Repo runtime | T035-T044 | Repo ADR, parity matrix, final Repo evidence |
| FR-011 public/private Repo operations | T037-T042 | authorization and ordinary-client negative tests |
| FR-012 raw payload adapts to exact Data | T036, T041-T044 | exact-packet fixtures and HA campaign |
| FR-013 canonical C++ stream state | T045-T051 | binding parity and UAV campaign |
| FR-014 retain UAV policy | T046, T048, T051 | UAV protocol/FEC/authority evidence |
| FR-015 static objects use large-data path | T049, T051 | forbidden-use test and DI/UAV smoke |
| FR-016 bounded typed compatibility | T052-T058 | mixed-version, typed-only, stored-state evidence |
| FR-017 no handler-less planners | T028, T034 | child 087 registry test |
| FR-018 semantic cache experimental | T028, T030, T034 | default-off boundary test |
| FR-019 retry requires idempotency | T014, T031 | child 085/087 retry tests |
| FR-020 test-gated removals | T008-T012, all child audits, T060-T067 | READY removal records and exact commands |
| FR-021 matched performance evidence | T004-T009, T032, T044, T051, T066 | frozen thresholds and raw matched campaigns |
| FR-022 independent rollback | T001, T010, every child acceptance, T069 | child rollback points |
| FR-023 umbrella/child ownership | T013, T019, T027, T035, T045, T052, T069 | six audited child features |

## Success Criteria

| Criterion | Evidence-producing tasks | Required artifact |
|---|---|---|
| SC-001 Core API has no app policy | T016-T018, T062 | `child-085-acceptance.md`, `final-core.md` |
| SC-002 zero V1/default placeholder paths | T023-T026, T028, T034 | child 086/087 scans and tests |
| SC-003 every lease tracked by its provider | T014-T018 | `child-085-acceptance.md` |
| SC-004 one Repo contract/runtime | T035-T044, T064 | Repo ADR and `final-repo.md` |
| SC-005 Repo HA invariants | T036, T042, T044, T064 | matched Repo raw results |
| SC-006 all focused regressions pass | T062-T065 | four final module evidence files |
| SC-007 UAV stream behavior preserved | T046-T051, T065 | child 089 parity and UAV results |
| SC-008 bounded typed-only migration | T053-T058 | child 090 mixed/typed-only evidence |
| SC-009 frozen performance gates pass | T009, T032, T044, T051, T066 | thresholds, raw runs, final comparison |
| SC-010 every removal has proof/rollback | T010, T060, T069 | READY removal-gate directory |
| SC-011 every child passes audit/acceptance | T017, T025, T033, T043, T050, T057, T069 | six audit and acceptance records |

Traceability is complete only when each child replaces planned evidence names
with actual commands, result paths, and commit/rollback identifiers.

## Supporting Task Coverage

The following program tasks support the primary mappings above and are listed
explicitly so no work exists without a requirement:

| Tasks | Requirements supported |
|---|---|
| T005, T006, T007 | FR-020, FR-021 baseline and reproducibility |
| T011 | FR-001, FR-008, FR-020 machine-checkable removal evidence |
| T021 | FR-002, FR-008 V2 authorization preservation |
| T038, T040 | FR-010, FR-011 Repo decision and reversible parity slices |
| T047 | FR-013 stream binding safety |
| T054, T055, T056 | FR-016 typed migration and semantic preservation |
| T059 | FR-022 conditional refactor isolation |
| T063 | FR-006, FR-007 final DI acceptance |
