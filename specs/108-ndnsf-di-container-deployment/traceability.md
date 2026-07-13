# Traceability: NDNSF-DI OCI Deployment Adapters

This is a pre-implementation map. Task and result paths are planned until their tasks execute. An unchecked task, missing artifact, failed live run, or deferred authority is never a PASS.

## Functional requirements

| Requirement | Design owner | Primary tasks | Acceptance surface |
|---|---|---|---|
| FR-001 immutable OCI manifest | Plan §1 | T017, T020, T025, T046 | release schema/digest tests |
| FR-002 runtime content, no secrets/routes | Plan §1, §8 | T035, T043-T047, T111-T112 | build context/layer scan |
| FR-003 common contract, two adapters | Plan §2 | T019, T027, T032-T033, T127 | adapter conformance matrix |
| FR-004 thin adapters/shared semantics | Plan §2 | T027-T031, T101, T127, T131 | interface and lineage tests |
| FR-005 preserve systemd rollback | Plan Constitution/Rollback | T003, T010, T126, T158 | systemd baseline/fallback |
| FR-006 Compose without host build | Plan §3 | T039-T052 | clean CPU canary |
| FR-007 one host-scoped NFD | Plan §3 | T037, T048-T051 | render/socket/readiness tests |
| FR-008 explicit cloud NFD routes | Plan §3/§6 | T053-T063 | port/route/remote invocation |
| FR-009 Docker GPU host prerequisites | Plan §3/§7 | T092-T094, T099 | driver/toolkit preflight |
| FR-010 Slurm allocation only | Plan §4 | T068-T069, T081, T087-T088, T123 | rendered job/process audit |
| FR-011 Apptainer, no Docker daemon/toolkit | Plan §4/§7 | T071, T080, T083, T095 | SIF/`--nv` tests |
| FR-012 OCI and SIF identity | Plan §4 | T026, T071, T080, T090 | dual-digest evidence |
| FR-013 named iTiger GRES | Plan §4 | T067, T073, T075-T077, T129 | three-class contract matrix |
| FR-014 explicit Slurm resources | Plan §4 | T068, T075-T081, T087 | deterministic `sbatch` render |
| FR-015 requested/allocated/physical/mapped GPU | Plan §4 | T073, T084, T090, T103 | GPU mapping evidence |
| FR-016 `--nv` and compatibility | Plan §4/§7 | T083, T092, T095, T100 | compute preflight/inference |
| FR-017 fail-closed runtime provider | Plan §7 | T004, T096-T102 | no-fallback negative |
| FR-018 fallback is degraded | Plan §7 | T098, T101-T102, T128 | degraded/outcome invariants |
| FR-019 `/home` small config only | Plan §5 | T066, T070, T078-T079 | storage policy negatives |
| FR-020 durable `/project` layout | Plan §5 | T066, T070, T075-T090 | project evidence promotion |
| FR-021 compute `/tmp` is scratch | Plan §5 | T066, T070, T082, T085-T086 | fsync and promotion tests |
| FR-022 actual capacity/quota validation | Plan §5 | T066, T070, T078-T079 | quota/shared-capacity tests |
| FR-023 unique run/promotion | Plan §5 | T031, T072, T074, T086-T090 | partial-copy/exactly-once tests |
| FR-024 multi-node iTiger network gate | Plan §6 | T014, T024, T132-T135 | measured two-node probe |
| FR-025 distinct external identities | Plan §8 | T104-T110, T114 | two-identity acceptance |
| FR-026 read-only/minimal secret mounts | Plan §8 | T105-T112 | mount and scan tests |
| FR-027 common complete evidence | Plan §2/§8 | T016, T028-T031, T101, T128, T131 | schema/mutation tests |
| FR-028 Compose evidence | Plan §3 | T037-T040, T050-T052, T063 | health/routes/release record |
| FR-029 Slurm evidence | Plan §4 | T069, T072-T090 | job state/resource/copy record |
| FR-030 full operator lifecycle | Plan §2-§4 | T018, T032-T033, T050, T087-T088, T119-T122 | CLI contract/lifecycle tests |
| FR-031 termination traps preserve exit | Plan §5 | T072, T081, T086 | signal/partial-promotion tests |
| FR-032 actionable fail-closed preflight | Plan §2/§7 | T014-T018, T056, T068-T074, T094-T100 | negative suite |
| FR-033 substrate separate from candidate | Plan §8 | T005, T029, T090, T103, T128 | authority fields |
| FR-034 production authority in Spec 106 | Scope/Plan §8 | T029, T128, T144-T145, T159 | forced DEFERRED contract |
| FR-035 full adapter test coverage | Validation Strategy | T011-T019, T035-T040, T053-T074, T094-T107, T127-T146 | offline/integration/live suites |
| FR-036 exact operator documentation | Quickstart/Phase 10 | T147-T154 | cloud/iTiger runbooks |

## Success criteria

| Criterion | Closing tasks | Planned evidence |
|---|---|---|
| SC-001 cloud CPU in 15 minutes | T039-T052 | `cloud-cpu` timing/readiness bundle |
| SC-002 two-host remote invocation | T053-T063 | `cloud-two-host` network/invocation bundle |
| SC-003 five-minute RTX 5000 substrate | T075-T090 | scheduler/SIF/GPU/scratch/project evidence |
| SC-004 parameterized RTX 6000/H100 | T076-T077, T129, T141-T142 | contract plus conditional live outcomes |
| SC-005 OCI and materialization identity | T017, T026, T071, T080, T128, T143 | digest lineage |
| SC-006 fail-closed/degraded GPU | T096-T103, T128 | negative/fallback/candidate bundle |
| SC-007 zero secret disclosure | T104-T114, T139 | OCI/SIF/evidence scan |
| SC-008 correct iTiger storage classes | T066, T070, T078-T090 | capacity/fsync/promotion evidence |
| SC-009 Compose rollback/Slurm cancellation | T115-T126 | lifecycle bundles |
| SC-010 unverified multi-node remains false | T014, T024, T132-T135 | probe or explicit BLOCK/DEFERRED |
| SC-011 cross-adapter contract coverage | T127-T146 | offline/integration/live summaries |
| SC-012 no false production PASS | T029, T128, T144-T145, T157, T159 | schema, mutation test, audit |

## User story coverage

| Story | Tasks | Independent evidence |
|---|---|---|
| US1 OCI/cloud CPU | T035-T052 | CPU OCI and single-host Compose bundle |
| US2 cloud multi-host | T053-T063 | route plus remote invocation bundle |
| US3 iTiger Slurm/Apptainer | T064-T090 | offline adapter suite and five-minute substrate bundle |
| US4 truthful backend | T091-T103 | fail-closed/degraded/candidate inference evidence |
| US5 identity/storage/security | T104-T114 | two-identity and zero-secret acceptance |
| US6 operations/recovery | T115-T126 | Compose rollback and Slurm cancel evidence |

## Authority and handoff

| Claim | Spec 108 maximum | Final owner |
|---|---|---|
| OCI reproducibility and adapter conformance | PASS | Spec 108 |
| iTiger scheduler/Apptainer/GPU/scratch substrate | PASS when measured | Spec 108 |
| Candidate-bound container inference/backend | PASS when measured | Spec 108 |
| Multi-node iTiger NFD networking | PASS only after T134; otherwise BLOCKED/DEFERRED | Spec 108 |
| Physical performance, real network/UAV, production security, soak | DEFERRED | Spec 106 |
