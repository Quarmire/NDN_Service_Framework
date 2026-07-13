# Spec 109 traceability

`PASS`, `FAIL`, `BLOCKED`, and `INCONCLUSIVE` below are evidence outcomes, not
task-completion markers. All task IDs have terminal implementation records; a
checked task may correctly preserve a failed or blocked experiment.

## Functional requirements

| Requirement | Tasks | Implementation / evidence | Outcome |
|---|---|---|---|
| FR-001 | T039,T052 | no-local/home sentinel, `storage-verdict.md` | PASS |
| FR-002 | T041,T049 | protected project layout and job scratch templates | PASS |
| FR-003 | T019,T029,T047-T048,T126-T132 | storage admission and capacity report | PASS gate behavior |
| FR-004 | T045,T053 | reference-aware cleanup plan | PASS |
| FR-005 | T045,T053 | cleanup requires dry-run | PASS |
| FR-006 | T011,T018,T028,T043-T051 | sealed 0.5B registry | PASS 0.5B |
| FR-007 | T038,T042-T044,T050-T051 | transfer/finalize jobs 146050/146123 | PASS 0.5B |
| FR-008 | T004,T013-T014,T106,T126 | seven-size fixtures and 105-cell matrix | PASS |
| FR-009 | T017,T020,T030,T067,T093,T109 | candidate/profile fingerprints and ledgers | PASS contracts |
| FR-010 | T021,T031,T107,T109-T125 | scoped ladder gates | PASS; all sizes systemic BLOCKED |
| FR-011 | T056,T064-T074 | oracle template/profile and terminal records | BLOCKED predecessor |
| FR-012 | T057,T062,T070-T073 | deterministic token contracts | BLOCKED live oracle |
| FR-013 | T025,T068-T069 | diagnostic identity separation | PASS contract; zero jobs |
| FR-014 | T080,T085-T090,T094-T104 | packaged security/orchestration | FAIL MiniNDN preflight; GPU BLOCKED |
| FR-015 | T002,T006-T007,T066,T089 | exact predecessor observation | BLOCKED 24/24 incomplete |
| FR-016 | T016-T017,T032,T089,T148 | adversarial validators | PASS 44 mutation tests |
| FR-017 | T023,T058,T078,T083-T084,T097 | node provider/GPU UUID gate | PASS tests; no GPU PASS |
| FR-018 | T023,T078,T083,T148 | fallback rejection | PASS tests |
| FR-019 | T024,T077,T079,T082,T092 | graph/tensor/KV checks | PASS offline; live BLOCKED |
| FR-020 | T024,T057,T070-T073,T079,T094-T097 | 1/2/32 contracts | PASS offline; live BLOCKED |
| FR-021 | T027,T059,T064,T067,T098-T100 | excluded warmup/60-second profile | PASS contract; live BLOCKED |
| FR-022 | T014,T021,T025,T098-T101,T109 | independent repetition ledger | PASS contract; zero runs |
| FR-023 | T027,T059,T101,T141,T145-T146 | metric/sample aggregation | UNAVAILABLE no runs |
| FR-024 | T027,T102,T142,T146 | critical-path reconciliation | PASS tests; live BLOCKED |
| FR-025 | T027,T030,T064,T093,T098-T103,T146 | matched comparison fingerprint | PASS contract; no pair |
| FR-026 | T016,T026,T031-T032,T148 | negative terminal evidence | PASS; failures retained |
| FR-027 | T022,T025,T031,T050,T070-T072,T094-T100 | exact-once ledger | PASS; measured jobs zero |
| FR-028 | T128,T130,T133-T139 | placement admission | one-node preferred; execution BLOCKED |
| FR-029 | T128,T130,T139 | `multinode-gate.json` | DEFERRED Spec108 T134 |
| FR-030 | T126-T133,T136 | large-model admission | PASS enforcement; zero transfer |
| FR-031 | T020,T030,T093,T109,T128-T130 | identity bindings | PASS tests |
| FR-032 | T022,T033,T042,T064,T087-T088 | bounded Slurm lifecycle | PASS templates/CLI |
| FR-033 | T026,T064,T087,T104 | trap/finalizer/original exit | PASS contract |
| FR-034 | T026,T144,T147,T154 | final secret scan | PASS zero findings |
| FR-035 | T016,T026,T032,T074,T097,T101-T104 | evidence schema/lineage | PASS contract; live BLOCKED |
| FR-036 | T016,T026,T032,T104,T148 | mutation cases | PASS 44/44 |
| FR-037 | T021,T031,T093,T109,T125,T140,T145 | finalized 105-cell matrix | PASS, 105 BLOCKED |
| FR-038 | T027,T032,T075,T105,T152,T155 | authority planes | PASS separation |
| FR-039 | T055,T075,T105,T113-T125,T132,T135,T138,T152 | negative reports | PASS retained |
| FR-040 | T157-T160,T165 | commands and handoff docs | PASS |
| FR-041 | T004,T018,T043,T051,T106 | license registry/fixtures | PASS 0.5B; others pre-transfer |
| FR-042 | T006,T032,T075,T105,T132,T155,T164 | Spec106 ownership | DEFERRED |
| FR-043 | T056-T075,T103,T146,T152 | oracle timing separation | PASS contract; oracle BLOCKED |
| FR-044 | T027,T064-T065,T093,T098-T103,T146,T152 | staged baseline denominator | UNAVAILABLE no matched pair |
| FR-045 | T013,T027,T030,T067,T093 | locked workload profile | PASS contract |
| FR-046 | T027,T059,T101,T143,T146,T150 | threshold/CI logic | PASS tests; metrics unavailable |
| FR-047 | T024,T057,T073,T077,T079,T092,T097,T143 | exact/numerical gates | PASS tests; live BLOCKED |
| FR-048 | T001,T008,T020,T030 | source snapshot | PASS sealed commit |
| FR-049 | T002,T007,T089 | predecessor lock/observation | BLOCKED exact entries |
| FR-050 | T007,T020,T030,T066-T067,T089 | deployment digest binding | BLOCKED Spec108 release |
| FR-051 | T017,T032,T035,T148 | Schema + semantic validator | PASS |
| FR-052 | T033-T034,T040,T046,T065,T088,T156 | repository CLI canonical | PASS |
| FR-053 | T014,T021,T031,T093,T109,T125,T140 | keyed cell ledger | PASS 105 terminal |
| FR-054 | T010,T163 | separate audit documents | PASS separation |

## Success criteria

| Criterion | Tasks | Evidence | Outcome |
|---|---|---|---|
| SC-001 | T039,T052 | storage-policy sentinel | PASS |
| SC-002 | T018,T028,T048,T051,T110-T123,T131-T138 | 0.5B registry plus pre-download gates | PASS |
| SC-003 | T056-T075 | `0.5b-reference-verdict.md` | BLOCKED |
| SC-004 | T076-T105 | `0.5b-candidate-verdict.md` | BLOCKED |
| SC-005 | T098-T101,T111-T123 | scale matrix | vacuous; no accepted size |
| SC-006 | T023,T078,T083,T097 | backend tests | PASS gate, no GPU PASS |
| SC-007 | T109,T125,T140,T145,T151 | 105/105 represented | PASS |
| SC-008 | T107,T109-T125 | ladder scoped-gate tests/records | PASS |
| SC-009 | T126-T138 | large-model admission | PASS enforcement; no starts |
| SC-010 | T027,T093,T103,T124,T146,T152 | matched overhead report | UNAVAILABLE, no mixing |
| SC-011 | T102,T142,T146 | critical-path test | PASS logic; no measured path |
| SC-012 | T144,T147,T154 | secret scan report | PASS zero findings |
| SC-013 | T016-T027,T035,T104,T148 | mutation suite | PASS 44/44 |
| SC-014 | T143,T149-T150 | reproduction verdict | INCONCLUSIVE, zero jobs |
| SC-015 | T128,T139 | multi-node gate | PASS, zero starts |
| SC-016 | T006,T032,T155,T164 | release gate/handoff | PASS deferred |
| SC-017 | T027,T103,T146,T152 | matched-overhead output | PASS invariant; no values |
| SC-018 | T023,T078,T083,T097 | node assignment validator | PASS gate; no GPU PASS |
| SC-019 | T027,T059,T101,T146 | sample thresholds | PASS logic; unavailable tails |
| SC-020 | T001-T002,T007-T008,T020,T030 | source/predecessor digests | source PASS, predecessor BLOCKED |
| SC-021 | T017-T027,T032,T035,T148 | contract/mutation suite | PASS |
| SC-022 | T021,T031,T107,T109-T125 | scoped gate behavior | PASS |

## Complete task/evidence accounting

| Task range | Terminal artifact |
|---|---|
| T001–T010 | `baselines/`, `code-reality.md`, pre-implementation audit |
| T011–T035 | fixtures, 30+ unit contracts, `offline-foundation/` |
| T036–T055 | discovery, transfer/seal jobs, storage verdict |
| T056–T075 | reference implementation plus predecessor-block records/verdict |
| T076–T090 | parameterized runtime, ORT evidence, Slurm orchestration, preflight |
| T091–T105 | finalized 0.5B blocked matrix and candidate verdict |
| T106–T125 | 1.5B–14B terminal ledgers and descriptive status table |
| T126–T140 | large-model admission, 32B/72B/multi-node gate records |
| T141–T155 | 105-cell aggregation, mutation/reproduction/secret scan, release gate |
| T156–T165 | operator docs, bilingual README, audits, Spec106 handoff, completion summary |

Ignored `results/` files are local mirrors. Durable accepted run evidence would
live under `/project/$USER/ndnsf-di/evidence`; no GPU run reached promotion.
