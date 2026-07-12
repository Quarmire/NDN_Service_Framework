# Telemetry Plan Validation

**Task**: T061  
**Date**: 2026-07-12  
**Scope**: deterministic MiniNDN controlled host telemetry  
**Physical GPU evidence**: false

## Executed Command

```bash
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference \
  python3 Experiments/spec105_telemetry_plan_cells.py
```

Exit status: 0. The executable fixture validates every expected final decision
before emitting its complete JSON record. It uses source
`controlled-minindn-host-probe`, CPU device `cpu0`, and no GPU measurement.

## Decision Summary

| Cell | Changed observation | Required limit | Decision | Reason |
|---|---:|---:|---|---|
| fresh | age 100 ms | <=2,000 ms | `reuse` | none |
| stale | age 2,001 ms | <=2,000 ms | `defer` | `TELEMETRY_STALE` |
| memory-pressure | free host memory 1,000 B | >=4,000 B | `defer` | `HOST_MEMORY_PRESSURE` |
| queue-pressure | ready queue 9 | <=2 | `defer` | `READY_QUEUE_PRESSURE` |
| membership-mismatch | `members-v2` | `members-v1` | `replan` | `MEMBERSHIP_VERSION_CHANGED` |
| device-mismatch | `cpu1` | `cpu0` | `reject` | `DEVICE_IDENTITY_MISMATCH` |

## Every Predicate Decision

`P` means PASS and `F` means FAIL. No predicate is omitted: all 18 predicates
returned by `evaluate_plan_feasibility()` are recorded for all six cells.

| Predicate | Fresh | Stale | Memory pressure | Queue pressure | Membership mismatch | Device mismatch |
|---|---:|---:|---:|---:|---:|---:|
| measured-source | P | P | P | P | P | P |
| freshness | P | F | P | P | P | P |
| provider-name | P | P | P | P | P | P |
| provider-boot | P | P | P | P | P | P |
| runner-kind | P | P | P | P | P | P |
| runtime-version | P | P | P | P | P | P |
| model-digest | P | P | P | P | P | P |
| plan-digest | P | P | P | P | P | P |
| device-id | P | P | P | P | P | F |
| membership-version | P | P | P | P | F | P |
| network-profile-version | P | P | P | P | P | P |
| cache-version | P | P | P | P | P | P |
| evidence-epoch | P | P | P | P | P | P |
| artifact-digests | P | P | P | P | P | P |
| free-host-memory | P | P | F | P | P | P |
| ready-queue | P | P | P | F | P | P |
| waiting-dependencies | P | P | P | P | P | P |
| active-workers | P | P | P | P | P | P |

## Shared Observations And Limits

Unless a cell's changed observation is listed above, all cells used these
observed/required pairs:

| Predicate | Observed | Required |
|---|---|---|
| measured-source | `controlled-minindn-host-probe`, `measured` | non-configured measured source |
| provider-name | `/provider/A` | `/provider/A` |
| provider-boot | `boot-a` | `boot-a` |
| runner-kind | `onnxruntime-cpu` | `onnxruntime-cpu` |
| runtime-version | `ort-1.26` | `ort-1.26` |
| model-digest | `sha256:model` | `sha256:model` |
| plan-digest | `sha256:plan` | `sha256:plan` |
| device-id | `cpu0` | `cpu0` |
| membership-version | `members-v1` | `members-v1` |
| network-profile-version | `network-v1` | `network-v1` |
| cache-version | `cache-v1` | `cache-v1` |
| evidence-epoch | 3 | >=3 |
| artifact-digests | `/Stage/0=sha256:stage0` | exact match |
| free-host-memory | 5,000 B | >=4,000 B |
| ready-queue | 1 | <=2 |
| waiting-dependencies | 1 | <=2 |
| active-workers | 1 | <=2 |

## Interpretation

The cells establish the required fail-closed routing: only fresh, matching,
measured host facts permit reuse; transient capacity pressure defers;
membership drift replans; device identity drift rejects. These are simulation
contract results, not throughput measurements or physical GPU validation.
