# Traceability: NDNSF-DI iTiger Qwen Scaling

## Functional requirements

| Requirements | Design/contract owner | Tasks | Closing evidence |
|---|---|---|---|
| FR-001, FR-002, FR-003, FR-004, FR-005 storage/path/cleanup | Plan §2 | T012,T019,T029,T036-T055 | discovery, admission, sealed layout, cleanup verdict |
| FR-006, FR-007, FR-008, FR-009 model/candidate identity | Data model model/candidate | T011,T018,T020,T028,T030,T043-T051 | sealed registry/candidate digests |
| FR-010 scoped progression | Plan §9, keyed matrix | T021,T031,T107,T109-T125 | scoped gate and continuation tests |
| FR-011, FR-012, FR-013 oracle/diagnostics | Plan §4 | T056-T075 | full-model oracle bundle |
| FR-014, FR-015, FR-016 runtime/security/prerequisites | Predecessor contract | T002,T006-T007,T080,T085-T090 | exact predecessor/security gate |
| FR-017, FR-018 truthful CUDA/fallback | Evidence Schema, semantic rules 8-10 | T023,T078,T082-T084,T097,T104 | node-provider profile and negatives |
| FR-019, FR-020 artifact/exact output | Plan §5 | T024,T077,T079,T091-T097 | tensor/KV/logit/token verdict |
| FR-021, FR-022, FR-023, FR-024 measurement/attribution | Plan statistics/§8 | T059,T098-T102,T141-T146 | original runs, valid tails, critical path |
| FR-025 matched controls | MatchedBaselinePair | T027,T030,T056,T064-T065,T098-T103 | comparison fingerprint and overhead |
| FR-026, FR-027 failures/no retry | Plan failure rules | T025-T026,T031-T033, all live cells | immutable terminal ledger |
| FR-028, FR-029, FR-030, FR-031 large/multi-node variants | Plan ladder | T126-T140 | admission/placement records |
| FR-032, FR-033, FR-034, FR-035 job/promotion/redaction/lineage | Slurm/evidence model | T022,T026,T032-T034,T042-T046,T144-T154 | scheduler/evidence bundle and scan |
| FR-036, FR-037, FR-038, FR-039, FR-040 fail-closed/matrix/reproduction | Semantic validator | T014,T016-T017,T025,T031-T035,T141-T153 | mutation, matrix, reproduction reports |
| FR-041, FR-042 license/physical authority | Authority model | T004,T018,T028,T043,T051,T106-T121,T148,T155,T164 | licenses and deferred release gate |
| FR-043, FR-044 separate oracle/baseline | Plan §§4,6 | T027,T056,T062,T064-T065,T098-T103,T111-T124 | zero oracle-timing overhead pairs |
| FR-045 workload identity | WorkloadProfile/Profile Schema | T013,T020,T027,T030,T056,T067,T093 | locked load/cache/run-order digest |
| FR-046 statistical validity | Plan statistics/Evidence Schema | T027,T059,T101,T143,T146,T150 | counts, CI, unavailable tails |
| FR-047 numerical equivalence | NumericalEquivalence | T024,T057,T073,T077,T079,T092,T097,T143 | hidden/KV/logit/token checkpoints |
| FR-048 source snapshot | SourceSnapshot Schema | T001,T008,T020,T030,T033,T035 | clean/sealed-dirty reproducibility |
| FR-049 exact predecessors | PredecessorGate Schema | T002,T007,T017,T032,T035,T089 | 24 keyed PASS entries and digests |
| FR-050 Spec 108 composition | DeploymentBinding | T007,T013,T020,T030,T066-T067,T089,T120 | profile/release digest resolution |
| FR-051 Schema plus semantic invariants | Contracts/semantic-validator.md | T014,T016-T017,T021-T027,T031-T035,T148 | adversarial behavioral probes |
| FR-052 repository-local authority | Operator CLI/quickstart | T033-T034,T040,T156-T160 | portable commands; Skill optional |
| FR-053 per-cell bundle closure | ScaleMatrix/ExperimentCell | T014,T021,T031,T093,T109-T125,T140-T151 | keyed cells/runs and partial-bundle tests |
| FR-054 separate audits | Audit checklists | T010,T161-T163 | pre- and post-implementation reports |

## Success criteria

| Criteria | Closing tasks | Evidence |
|---|---|---|
| SC-001, SC-002 storage/model registry | T039,T043-T055,T110-T140 | storage policy, registry, rejected admission |
| SC-003, SC-004 oracle/candidate correctness | T068-T075,T089-T097,T105 | exact and numerical 0.5B verdicts |
| SC-005, SC-006 repetitions/CUDA | T078,T083,T097-T104,T112/T115/T118/T122/T134/T137 | original repetitions and node-level CUDA |
| SC-007, SC-008, SC-009 terminal matrix/admission | T021,T031,T109-T140,T145-T151 | keyed terminal matrix |
| SC-010, SC-011 overhead/attribution | T027,T098-T103,T124,T142,T146,T152 | matched staged overhead and critical path |
| SC-012, SC-013 secrets/mutations | T016-T017,T026,T032,T104,T144,T147-T148,T154 | fail-closed mutation/scan results |
| SC-014 reproduction | T143,T149-T150 | exact/numerical/CI verdict |
| SC-015, SC-016 network/physical | T128,T130,T139-T140,T155,T164 | network gate and Spec 106 handoff |
| SC-017 oracle timing excluded | T027,T103,T124,T146,T152 | comparison-role audit |
| SC-018 node-level CUDA | T023,T078,T083,T097,T104 | complete ORT assignment record |
| SC-019 sample-qualified tails | T027,T059,T101,T143,T146 | counts and unavailable markers |
| SC-020 source/predecessor reconstruction | T001-T002,T007-T008,T032,T035,T089 | content reconstruction report |
| SC-021 adversarial contracts | T014,T016-T017,T021-T027,T032,T035,T148 | rejected duplicate/contradictory fixtures |
| SC-022 local failure continuation | T021,T031,T107,T109-T125 | scoped-gate continuation report |

## User stories and authority

| Story | Tasks | Independent closure |
|---|---|---|
| US1 storage-safe staging | T036-T055 | sealed 0.5B plus cleanup/path verdict |
| US2 oracle and baseline inputs | T056-T075 | full-model tokens, numerical export inputs, workload lock |
| US3 real 0.5B candidate | T076-T105 | secured exact/numerical candidate plus three matched pairs |
| US4 1.5B-14B | T106-T125 | keyed terminal cells, scoped gates, descriptive/controlled tables |
| US5 32B/72B | T126-T140 | pre-download admission and conditional outcomes |
| US6 reproducibility | T141-T155 | complete matrix, mutation, CI reproduction, reports |

Substrate is owned by Spec 108; full-model oracle, artifact, staged baseline, and iTiger candidate evidence by Spec 109; generation/security semantics by Spec 107; physical production by Spec 106.
