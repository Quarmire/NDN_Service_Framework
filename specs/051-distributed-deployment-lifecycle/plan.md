# Implementation Plan: Distributed Deployment Lifecycle

> **Superseded; do not execute.** Accepted implementation and evidence are in
> Specs 085 and 087. This file is retained only to explain the discarded draft.

**Branch**: `051-distributed-deployment-lifecycle` | **Date**: 2026-07-06 |
**Spec**: [spec.md](spec.md)

## Architecture

```
User A ──deploy(plan)──▶ NDN Interest multicast
                             │
                ┌────────────┼────────────┐
                ▼            ▼            ▼
             /P/a ACK     /P/b ACK     /P/c ACK
            "Backbone,   "Head/0,     "Backbone,
             cost=8ms"    cost=0ms"    cost=35ms,
                                       provisioning
                                       dep-abc, 5s ETA"
                │            │            │
                └────────────┼────────────┘
                             │
                ┌────────────▼────────────┐
                │  User A 自己做决策       │
                │  Filter → Score → Pick   │
                │  Reserve leases          │
                └─────────────────────────┘
                             │
                ┌────────────▼────────────┐
                │  NDNSD 广播 deployment   │
                │  → User B 立即发现       │
                │  → User B 可以直接执行   │
                └─────────────────────────┘
```

## Boundary: NDNSF Core vs NDNSF-DI

NDNSF Core owns:
- `PlacementConstraint` (service-neutral resource constraints)
- `Deployment` dataclass (generic lifecycle: PROVISIONING/ACTIVE/DEGRADED/DISK_RESIDENT/EVICTED)
- `deploy_service` / `discover_deployments` API shape
- NDNSD deployment broadcasting helpers

NDNSF-DI owns:
- Fragment-aware filtering (`min_gpu_memory_mb`, `required_backend`)
- DI-specific placement scoring (residency cost, edge cost)
- `fragment_map` structure (role → provider mapping)
- Model-specific provisioning logic

## Key Files

| File | Changes |
|---|---|
| `NativeProviderReadiness.hpp/cpp` | +`setProvisioningContext`, +ACK fields |
| `DI_NativeProviderExecutable.cpp` | Call `setProvisioningContext` at provisioning start |
| `pythonWrapper/ndnsf/runtime_telemetry.py` | +`PlacementConstraint`, +`DeploymentStatus` |
| `pythonWrapper/ndnsf/service.py` | +`deploy_service`, +`discover_deployments`, +`deployment_id` in `request_collaboration` |
| `runtime_v1.py` | +`Deployment`, +`filter_feasible_providers`, +`score_provider_candidates`, +`pick_optimal_assignment` |
| `user_driver.py` | Parse new ACK fields, +`--deployment-id` |
| `advisory_coordinator.py` | Read NDNSD deployment state for health tracking |

## No Changes To

- `coordination.py` (CoordinationIntent/Suggestion unchanged)
- `ServiceProvider.hpp/cpp` (already has NDNSD infrastructure)
- NDN wire protocol (all deployment data flows through existing ACK payload + NDNSD meta)

## Validation

```bash
# Unit tests
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference python3 tests/python/test_ndnsf_deployment.py
# MiniNDN deploy→execute smoke
sudo -n PYTHONPATH=... python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --deployment-mode --advisory-coordinator ...
```
