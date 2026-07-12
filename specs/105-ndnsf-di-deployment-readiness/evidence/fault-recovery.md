# Fault Recovery Matrix

**Tasks**: T076, T078  
**Date**: 2026-07-12  
**Result**: **BLOCK**  
**Physical hardware evidence**: false

## Frozen Execution

```bash
PYTHONPATH=pythonWrapper:NDNSF-DistributedInference \
  python3 Experiments/NDNSF_DI_LlmPipeline_Minindn.py \
    --fault-matrix-contract \
    --output-dir results/spec105-fault-recovery-20260712T120900Z
```

The expected exit status was 2 and was retained in `outer-exit-code.txt`.
Machine-readable output is
`results/spec105-fault-recovery-20260712T120900Z/fault-matrix-contract.json`.
No cell was deleted or repeated.

## Scope Boundary

The hook executes inside the MiniNDN harness source and freezes the recovery
contract, same-three-node fallback map, control payloads and expected terminal
authority. It explicitly records `networkInjection=false`. It does **not**
kill live MiniNDN provider processes or corrupt live segmented Data. Therefore
the contract checks below are useful implementation evidence but cannot close
the live fault-recovery gate. This limitation controls the overall `BLOCK`.

## Same-Three-Node Fallback Map

| Role | Primary | Fallback |
|---|---|---|
| `/LLM/Pipeline/Stage/0` | UCLA | Arizona |
| `/LLM/Pipeline/Stage/1` | Arizona | WUSTL |
| `/LLM/Pipeline/Stage/2` | WUSTL | UCLA |

No fourth node or physical host is introduced.

## Retained Cells

| Cell | Contract outcome | Attempt/terminal evidence | Live network proof |
|---|---|---|---|
| provider kill/restart | one replacement | epoch 1 cancelled/superseded; epoch 2 selected; old output rejected | MISSING |
| straggler | one replacement | 3,900 ms remaining; old output rejected | MISSING |
| missing segment | fail | `DEPENDENCY_MISSING` | MISSING |
| dependency hash mismatch | fail | `DEPENDENCY_HASH_MISMATCH` | MISSING |
| stale telemetry | one replacement | epoch 2 selected; old output rejected | MISSING |
| cache eviction | retry full context | same provider, epoch 2, full-context required | MISSING |
| provider restart/new boot | one replacement | old boot attempt superseded | MISSING |
| late old output | reject old | `oldEpochAuthoritative=false` | MISSING |

Every recoverable cell contains both `CANCEL` and `SUPERSEDE` under schema
`ndnsf-di-execution-control-v1`, transported as an existing DI service payload.
No new Core wire name appears.

## Distributed Fallacy Scan

The contract includes explicit checks for all 11 project fallacies:

| Fallacy | Contract coverage | Network evidence |
|---|---|---|
| network is reliable | failure reason retained | BLOCK |
| latency is zero | remaining deadline retained | BLOCK |
| bandwidth is infinite | dependency fault cell present | BLOCK |
| network is secure | authenticated attempt/control schema required | BLOCK |
| topology is static | fallback map changes authority | BLOCK |
| one administrator | provider identity retained | BLOCK |
| transport cost is zero | no performance claim | BLOCK |
| network is homogeneous | role/provider mapping explicit | BLOCK |
| time is synchronized | monotonic remaining deadline used | BLOCK |
| resources are stable | stale/cache/restart cells present | BLOCK |
| failures are independent | exact per-attempt authority retained | BLOCK |

The machine-readable `fallacyScan` is 11/11 PASS for contract presence only.
All 11 remain BLOCK for live network evidence.

## Decision

The bounded recovery implementation is testable and its negative outcomes are
retained. The live MiniNDN fault campaign is not proven. Final Spec 105 release
evidence must carry this `BLOCK`; neither T076/T078 completion nor the 11/11
contract scan may be interpreted as deployment acceptance.
