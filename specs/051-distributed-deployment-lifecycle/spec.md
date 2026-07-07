# Spec: Distributed Deployment Lifecycle

**Branch**: `051-distributed-deployment-lifecycle` | **Date**: 2026-07-06 |
**Status**: Draft

## Summary

Separate model deployment from request execution. A Deployment is a system-global,
shared resource — any User can discover and use it. Placements are decided by
Users (not a single Coordinator), using the existing ACK/Selection/Lease mechanism.
Deployment state converges across all nodes via NDNSD SVS gossip.

## Key Design Decisions

1. **Deployment is global, not per-user.** User A's deployment is visible to
   User B via NDNSD. Both can execute against the same deployed model.

2. **User makes placement decisions.** User sends deploy Interest → providers ACK
   with capacity + provisioning context → User selects optimal providers →
   reserves with leases. Coordinator is advisory only.

3. **NDNSD broadcasts deployment state.** After User locks in a deployment,
   the result is published via NDNSD as meta info. All nodes converge to the
   same view through SVS gossip (deterministic tie-breaking).

4. **Provider negative-ACK carries provisioning context.** During provisioning,
   a provider tells other users: "I'm loading deployment X, role Y, ETA Z ms."
   This lets other users decide whether to wait or pick another provider.

5. **DISK_RESIDENT is a valid cache state.** Models on disk but not in memory
   are not "expired" — they have a known readyCost (35ms). Users can prefer
   ACTIVE deployments but fall back to DISK_RESIDENT.

## Deployment Lifecycle

```
PROVISIONING ──(all providers loaded)──▶ ACTIVE
     │                                      │
     │ (partial failure)              (evict from memory)
     ▼                                      ▼
DEGRADED                              DISK_RESIDENT
     │                                      │
     │ (recover)                      (reload)
     ▼                                      │
ACTIVE ◀────────────────────────────────────┘
     │
     │ (evict from disk)
     ▼
EVICTED
```

## User Scenarios

### P1: Deploy a model and execute against it

**Given** no deployment exists, **when** User A calls `deploy_service(plan)`,
**then** providers ACK with capacity, User A selects optimal ones and reserves
leases, deployment goes ACTIVE, NDNSD broadcasts it, and User A can execute
`request_collaboration(deployment_id=...)` with minimal latency.

### P2: Discover existing deployments

**Given** deployment dep-abc is ACTIVE, **when** User B calls `discover_deployments(service)`,
**then** User B sees dep-abc with fragment_map and status=ACTIVE, and can
immediately execute against it without re-deploying.

### P3: Provider provisioning informs other users

**Given** /P/a is loading model for dep-abc, **when** User C sends Interest,
**then** /P/a negative-ACKs with `reason=PROVISIONING`, `deploymentId=dep-abc`,
`provisioningRole=/Backbone`, `expectedReadyMs=5000`. User C can wait or pick
another provider.

### P4: DISK_RESIDENT fallback

**Given** dep-abc is DISK_RESIDENT (all providers evicted from memory),
**when** User D executes against dep-abc, **then** providers reload from disk
(35ms ready cost) and deployment transitions back to ACTIVE.

## Requirements

- REQ-051-001: `ServiceUser.deploy_service(plan, constraints)` MUST send deploy
  Interest, collect provider ACKs with provisioning proposals, let User select
  optimal providers, and reserve leases.
- REQ-051-002: Providers MUST include `deploymentId`, `provisioningRole`,
  `expectedReadyMs` in negative-ACK payload when in provisioning state.
- REQ-051-003: deployment state MUST be published via NDNSD `serviceMetaInfo`
  as a JSON field `deployments`.
- REQ-051-004: `ServiceUser.discover_deployments(service)` MUST return all
  deployments visible via NDNSD for that service, with status and fragment_map.
- REQ-051-005: `request_collaboration` MUST accept optional `deployment_id`
  parameter that routes to the providers in the deployment's fragment_map.
- REQ-051-006: Deployment status transitions MUST be deterministic from the
  same set of NDNSD observations (SVS convergence).
- REQ-051-007: `DISK_RESIDENT` MUST be treated as a valid cache state with
  `readyCost=35ms`, distinct from EVICTED.

## Non-Goals

- Coordinator-as-authority (coordinator remains advisory, optional cache)
- Auto-scaling / auto-eviction policies (manual deploy/evict only for now)
- Cross-service deployment (one deployment per service)
