# Distributed execution state contract

## State transitions

| State | Required evidence | May close live task? |
|---|---|---|
| `PLANNED` | candidate/cell key | No |
| `PREFLIGHT_BLOCKED` | blocker code, command, original output | No |
| `READY_TO_SUBMIT` | all admission digests and render checksum | No |
| `SUBMITTED_NOT_STARTED` | job ID and Slurm state | No |
| `CANDIDATE_EXECUTION_STARTED` | secured request plus candidate-bound GPU stage-start proof | No |
| `EXECUTED_PASS` | placement-specific dataflow, exact output, complete evidence | Yes |
| `EXECUTED_FAIL` | admissible post-start failure boundary and complete evidence | Yes |
| `EVIDENCE_INCOMPLETE` | known execution with missing/corrupt promotion | No |

## Execution boundary

`CANDIDATE_EXECUTION_STARTED` requires all of:

1. candidate/run/cell and Slurm allocation identifiers;
2. a secured NDNSF request accepted into a generation session;
3. a provider role beginning actual model execution;
4. provider PID/identity, node, NFD, stage, backend, and allocated GPU UUID correlation;
5. a timestamp after readiness and before the recorded terminal event.

Preflight, model download, export, NFD ping, ACK-only exchange, stage readiness,
GPU visibility, and standalone inference do not reach this boundary.

## Placement-specific PASS invariants

`EXECUTED_PASS` additionally requires:

- three expected stage roles executed by distinct providers/GPUs under the frozen candidate;
- `single-node-multi-gpu`: exactly one node/NFD and no claimed cross-node edge;
- `multi-node`: at least two nodes/NFDs and one cross-node dependency edge;
- no CPU fallback or unbound artifact;
- exactly one terminal response;
- exact output tokens equal to the oracle;
- atomic, checksum-valid durable evidence promotion.

## Failure and replacement

- Failures before the boundary leave the live task open.
- Failures after the boundary are immutable `EXECUTED_FAIL` negatives with
  `failureBoundary` equal to `stage-load`, `stage-execution`,
  `dependency-transfer`, or `terminal-response`.
- A FAIL proves only the recorded boundary; it cannot be narrated as a complete
  dataflow or PASS.
- No automatic Slurm resubmission is permitted. Bounded provider replacement
  inside the same generation session follows its frozen attempt/lease contract
  and is recorded separately from job retry.
- Human-approved replacement creates a new run/cell identity and a
  `replacesRunId` link; it never edits or hides the original.
