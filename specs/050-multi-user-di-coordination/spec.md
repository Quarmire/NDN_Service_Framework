# Spec: Multi-User DI Coordination Hardening

**Feature Branch**: `050-multi-user-di-coordination` | **Date**: 2026-07-06 |
**Status**: Implemented

## User Scenarios

### P1: Fragment-Aware Provider Selection

**Given** providers have different model fragment residency states (GPU_LOADED,
CPU_RESIDENT, DISK_RESIDENT, MISSING), **when** the advisory coordinator suggests
role assignments, **then** providers with ready fragments (low readyCostMs) are
preferred over providers that need cold-start loading.

### P2: Coordinator State Persistence

**Given** the coordinator has accumulated provider usage history,
**when** the coordinator process restarts, **then** the rolling fairness state is
recovered from a JSON file, avoiding the first post-restart requests all landing
on the same provider.

### P3: Intent Priority

**Given** multiple users with different priority levels, **when** `--enable-priority`
is set, **then** intents with higher `utility_weight` get first pick of providers.

## Requirements

- REQ-050-001: Coordinator MUST read `fragmentState` from intent payload and merge
  per-provider-per-role fragment residency into a rolling state table.
- REQ-050-002: Coordinator MUST apply `fragment_ready_ms` penalty in provider scoring
  based on observed residency (GPU_LOADED=0ms, CPU_RESIDENT=8ms, DISK_RESIDENT=35ms,
  REPO_AVAILABLE=120ms, MISSING=1,000,000ms).
- REQ-050-003: MiniNDN harness MUST write `fragment-inventory.json` from real provider
  inventory and pass it to the user driver via `--fragment-inventory-json`.
- REQ-050-004: Coordinator MUST persist `provider_use`, `provider_available_at_ms`,
  and `window_version` to a JSON state file after each window.
- REQ-050-005: Coordinator MUST load persisted state on startup, ignoring entries
  older than `--state-ttl-ms`.
- REQ-050-006: When `--enable-priority` is set, intents MUST be sorted by
  `(-utility_weight, created_at_ms, intent_id)`.

## Non-Goals

- Provider-to-coordinator direct telemetry (still via user ACK observations)
- Coordinator HA/failover (single instance)
- Cross-provider load balancing at the provider level
- C++ changes

## Success Criteria

- All existing tests pass without regression
- MiniNDN sequential smoke: 4 requests, all SUCCESS, fragment state logged
- MiniNDN RPS sweep: pure vs advisory stable up to 0.8 RPS
