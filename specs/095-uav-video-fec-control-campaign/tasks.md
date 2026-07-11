# Tasks: UAV Video FEC And Control Campaign

## Phase 1: Baseline And Design

- [x] T001 [US1] Inventory video start fields, Drone FEC ownership, launcher modes, campaign metrics, tests, and current Spec 089 evidence.
- [x] T002 [US2] Freeze the primary treatment matrix, constants, acceptance gates, output schema, timeout, and no-retry rule.
- [x] T003 [US3] Record baseline build/tests, disk, CodeGraph status, and existing 5% campaign evidence.
- [x] T004 [US1] Run strict Spec Kit structure/analyze and pre-implementation audit; resolve blockers.

## Phase 2: FEC Treatment Control

- [x] T005 [US1] Add validated Ground Station parity configuration and video request propagation.
- [x] T006 [US1] Add Drone parity parsing/default/reporting and preserve data-only StreamChunk publication when parity is zero.
- [x] T007 [US1] Add focused C++ tests for default, zero, one, and invalid parity values.
- [x] T008 [US1] Build and run affected UAV/Core stream tests.

## Phase 3: Canonical Campaign

- [x] T009 [US2] Extend the MiniNDN launcher with parity forwarding and concurrent video plus MAVLink acceptance checks.
- [x] T010 [US2] Refactor the parity campaign into deterministic loss/parity/repetition cells with generated matched topologies.
- [x] T011 [US3] Expand parsing for final decoded count, control outcomes, duration, malformed metrics, and accepted parity.
- [x] T012 [US3] Add treatment aggregation and stable JSON/per-run CSV/treatment CSV outputs.
- [x] T013 [US2] Add Python tests for commands, matrix size, parsing, aggregation, and failure gates.
- [x] T014 [US2] Run campaign dry-run and focused launcher/campaign tests.

## Phase 4: MiniNDN Evidence

- [x] T015 [US2] Run the 12-run primary MiniNDN campaign with 60-second streams and no automatic retry.
- [x] T016 [US3] Validate every run's video, control, FEC, latency, gap, stale, and bounded-buffer evidence.
- [x] T017 [US3] Compare matched cells descriptively and run the full affected regression suite.

## Phase 5: Closure

- [x] T018 [US3] Write reproducibility, result, negative-result, limitation, and residual-risk evidence.
- [x] T019 [US3] Run post-implementation Spec Kit analyze/audit/converge and execute any appended task.
- [x] T020 [US3] Update GSD/agent context/CodeGraph, verify no proposal changes and clean git, then close.

## Phase 6: Convergence

- [x] T021 [US3] Reject missing or malformed mandatory adaptive metrics and preserve field-level diagnostics per FR-009 (partial).
