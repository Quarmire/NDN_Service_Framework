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

## Final Exact Evidence Index

Command identifiers below are literal reproducible commands or command groups:

| ID | Exact command |
|---|---|
| C1 | `./waf build --targets=unit-tests -j2 && ./build/unit-tests --log_level=message` |
| C2 | `python3 -m unittest discover -s tests/python -p 'test_*.py'` |
| C3 | `sudo -n timeout 600s examples/run_security_regressions.sh` |
| C4 | `python3 Experiments/NDNSF_Transfer_Boundary_Documentation_Regression.py` |
| C5 | `sudo -n timeout 300s python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --runtime-profile examples/di-native-tracer.runtime.json --out results/spec090-typed-envelope/typed-only --requests 1 --concurrency 1 --provider-check-timeout 60 --no-local-execution-only --full-network` |
| C6 | Run `sudo -n -E python3 -B Experiments/NDNSF_NewAPI_Minindn_Perf.py --topology-file Experiments/Topology/AI_Lab.conf --controller-node memphis --user-node memphis --provider-nodes ucla --providers 1 --duration 35 --warmup 7 --interval-ms 500 --max-requests 10 --strategy first-responding --ack-timeout-ms 200 --timeout-ms 3000 --nlsr-converge-seconds 2 --controller-settle-seconds 2 --provider-start-gap-seconds 0 --post-ready-settle-seconds 2 --startup-settle-seconds 1 --provider-ready-timeout-seconds 30 --nfd-log-level ERROR` once normally and once with `--targeted` |
| C7 | `sudo -n timeout 300s python3 Experiments/NDNSF_DistributedRepo_Generic_Minindn.py --output-dir results/spec084-final/repo-integration --ha-campaign --campaign-duration-s 10 --campaign-rps 0.5 --campaign-concurrency 4 --campaign-read-ratio 0.8 --campaign-object-bytes 4096 --campaign-object-mode exact --campaign-replication-factor 2 --campaign-write-consistency ALL --campaign-control-mode targeted --campaign-request-timeout-ms 30000 --campaign-fail-repo repoA --campaign-fail-at-s 3 --campaign-restart-after-s 3 --campaign-auto-repair --campaign-repair-workers 2 --campaign-repair-max-jobs 4 --campaign-seed 88401` |
| C8 | `timeout 240s python3 Experiments/NDNSF_UAV_Stream_Parity_Campaign.py --out results/spec084-final/uav-stream-loss5 --runs 1 --auto-stop-seconds 8` |
| C9 | In independent detached worktrees run `git revert --no-commit 3918c98`, `git revert --no-commit b3acfd1`, `git revert --no-commit 00e4709`, `git revert --no-commit 5aca321`, `git revert --no-commit 01466f5`, `git revert --no-commit 72dc052`, or `git revert --no-commit f714c99`, followed by the focused regression named in each acceptance record |
| C10 | `codegraph sync . && codegraph explore "final architecture boundary after Specs 085-090..."` |

### Functional Requirements

| Requirement | Child/task | Commands | Exact evidence |
|---|---|---|---|
| FR-001 | 085 T034-T038; 084 T059-T061 | C10 | `evidence/final-structural-decision.md`, `evidence/removal-gates/final-gates.md` |
| FR-002 | 085/086/088/089/090 acceptance | C1,C3 | `evidence/final-core.md` and all six child acceptance files |
| FR-003 | 085 T007-T030 | C1,C5 | `evidence/child-085-acceptance.md` |
| FR-004 | 085 T020-T030 | C2,C5 | `specs/085-core-boundary-fail-closed-leases/evidence/minindn-acceptance.md` |
| FR-005 | 085 T031-T038; 087 | C2,C10 | `evidence/final-adversarial-review.md` |
| FR-006 | 085/087 | C2,C5 | `evidence/final-di.md` |
| FR-007 | 087 retention gate; `f714c99` | C5,C9 | `evidence/child-087-acceptance.md`, `evidence/final-core.md` |
| FR-008 | 086 T007-T026 | C1,C3,C6,C9 | `evidence/child-086-acceptance.md` |
| FR-009 | 086 | C1,C3,C6 | `specs/086-v2-invocation-permission-migration/evidence/minindn-acceptance.md` |
| FR-010 | 088 | C1,C2,C7 | `evidence/child-088-acceptance.md` |
| FR-011 | 088 | C2,C7 | `specs/088-repo-canonical-runtime-contract/evidence/private-operation-authorization.md` |
| FR-012 | 088 | C1,C2,C7 | `specs/088-repo-canonical-runtime-contract/evidence/frozen-fixtures.md` |
| FR-013 | 089 | C1,C2,C8 | `evidence/child-089-acceptance.md` |
| FR-014 | 089 | C1,C2,C8 | `evidence/final-uav.md` |
| FR-015 | 089/090 | C4,C5,C8 | `evidence/final-di.md`, `evidence/final-uav.md` |
| FR-016 | 090 | C1,C2,C5,C9 | `evidence/child-090-acceptance.md` |
| FR-017 | 087 | C2 | `evidence/child-087-acceptance.md` |
| FR-018 | 087 | C2 | `specs/087-di-policy-lifecycle-isolation/evidence/implementation-status.md` |
| FR-019 | 085/087 | C1,C2 | `evidence/child-085-acceptance.md`, `evidence/child-087-acceptance.md` |
| FR-020 | all children; 084 T060-T067 | C1-C10 as applicable | `evidence/removal-gates/final-gates.md` |
| FR-021 | 085-090 frozen gates | C5-C8 | `evidence/final-occam-report.md` |
| FR-022 | all children and `f714c99` | C9 | all child acceptance files plus `evidence/final-core.md` |
| FR-023 | 085-090 | C9 | `contracts/child-feature-map.md` and six child acceptance files |

### Success Criteria

| Criterion | Commands | Result and evidence |
|---|---|---|
| SC-001 | C2,C10 | No public app policy; `final-adversarial-review.md` |
| SC-002 | C1,C10 | Zero active V1/default placeholder path; child 086/087 acceptance |
| SC-003 | C1,C5 | Provider-local fail-closed leases; child 085 acceptance |
| SC-004 | C1,C2,C7 | One Repo network adapter; `final-repo.md` |
| SC-005 | C7 | Three 30/30 RF=2/W=ALL campaigns; `final-repo.md` |
| SC-006 | C1-C8 | All focused regressions pass; `final-core.md` through `final-uav.md` |
| SC-007 | C1,C2,C8 | 3/3 loss campaign, zero frame gap; `final-uav.md` |
| SC-008 | C1,C2,C5 | Typed-only and mixed-reader 2/2; child 090 acceptance |
| SC-009 | C5-C8 | Frozen gates pass or negative result triggers removal; `final-occam-report.md` |
| SC-010 | C9 | Every removal READY/deferred with owner/expiry; `final-gates.md` |
| SC-011 | C1-C10 | All six children PASS; `child-085-acceptance.md` through `child-090-acceptance.md` |
