# Tasks: Multi-User DI Coordination Hardening

## Phase 1: Fragment State in Coordinator

- [x] T001 Add `RESIDENCY_READY_COST_MS` constant and `--fragment-ready-default-ms` / `--fragment-state-ttl-ms` args to `advisory_coordinator.py`.
- [x] T002 Add `_merge_fragment_state()` and `_fragment_ready_penalty()` helper functions.
- [x] T003 Add `fragment_state_table` to coordinator state; merge `fragmentState` from intent payload; include `fragment_ready_ms` in scoring.
- [x] T004 Log `NDNSF_DI_ADVISORY_COORDINATOR_FRAGMENT_STATE` when fragment state is received; include in score_breakdown.

## Phase 2: Harness Fragment Inventory Feed

- [x] T005 Add `--fragment-inventory-json` flag to `user_driver.py` with `load_fragment_inventory()` helper.
- [x] T006 Add `fragment_inventory_json` parameter to `user_driver_command()` in harness; write `fragment-inventory.json` after provider inventory is collected.
- [x] T007 Pass `--fragment-inventory-json` in child process and process-pool command builders.

## Phase 3: Coordinator State Persistence

- [x] T008 Add `--state-file` and `--state-ttl-ms` args; implement `_load_state()` and `_save_state()`.
- [x] T009 Load persisted state in `make_handler()`, save after each `handle()` call; log `STATE_LOADED`.

## Phase 4: Intent Priority

- [x] T010 Add `--enable-priority` flag; sort intents by `(-utility_weight, created_at_ms, intent_id)` in `handle()`.

## Phase 5: Tests

- [x] T011 Update test assertion for new `advisoryMode` string.
- [x] T012 Run all existing tests (31 tests pass).
- [x] T013 Dry-run harness and RPS sweep commands verified.

## Phase 6: MiniNDN Validation

- [x] T014 Sequential smoke: 4 requests, all SUCCESS, fragment state received 4x.
- [x] T015 RPS sweep: pure vs advisory stable to 0.8 RPS, fragment state delivered to all requests.
