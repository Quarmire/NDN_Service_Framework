# Traceability: Spec 107

| Requirement | Primary tasks | Verification |
|---|---|---|
| FR-001–FR-002 | T001–T002, T009, T072 | Frozen lineage digests and path write denial |
| FR-003–FR-005 | T003–T004, T021, T024 | Candidate/campaign identity and diagnostic ineligibility tests |
| FR-004 | T003–T004, T024, T035, T046, T058 | Separate campaign classes, identities, roots, and eligibility labels |
| FR-006–FR-007 | T012, T018, T021–T027 | Reconciliation coverage and unique 25% branch gate |
| FR-008 | T015–T017, T031–T034, T072 | Application-layer codec plus Core-boundary audit |
| FR-009–FR-010 | T029, T035–T038 | Frozen profile and exact 1/2/32-token checks |
| FR-011–FR-012 | T011, T018, T022–T023, T028 | Bound tests and stable sampled timelines |
| FR-013–FR-014 | T035, T039–T042 | Three once-only 60-second per-repetition verdicts |
| FR-015–FR-016 | T005–T008, T037, T063 | Content-addressed store and disk/output preflight |
| FR-017–FR-020 | T013, T043–T053 | Owned live injection, one replacement, authority and cleanup |
| FR-018 | T046, T048–T053 | Positive control plus all eight once-only live-fault cells |
| FR-019 | T043–T053 | One replacement, original deadline, supersession, one terminal authority |
| FR-021–FR-023 | T054–T065 | Local supervision, operations, gated 24-hour soak |
| FR-022 | T055, T058, T062, T065 | Repo preservation and identity-incompatible cache disposal |
| FR-024 | T030, T067–T068 | Security negatives and bundle secret/content scan |
| FR-025–FR-026 | T014, T066–T072 | Mechanical local gate, predecessor BLOCK, physical DEFERRED |
| SC-001 | T009, T072 | Five frozen identifiers and zero predecessor changes |
| SC-002 | T025–T027 | At least 99% timing reconciliation and one branch |
| SC-003 | T038, T042 | Exact tokens and coherent provider evidence |
| SC-004 | T039–T042 | Independent completion, throughput, and p95 gates |
| SC-005–SC-006 | T048–T053 | Eight injected cells and bounded cleanup |
| SC-007–SC-008 | T060–T062, T065 | Two canaries and Repo-safe upgrade/rollback |
| SC-009 | T063–T065 | One eligible unreplaced 24-hour soak |
| SC-010 | T005–T008, T035, T063 | Preflight, no ENOSPC/reuse/re-export |
| SC-011 | T066–T069 | Digest-bound PASS/BLOCK and physical DEFERRED |

Every functional requirement and success criterion has at least one test or
evidence-producing task. Campaign tasks additionally inherit the no-retry,
unique-output, and negative-result retention rules in `tasks.md`.
