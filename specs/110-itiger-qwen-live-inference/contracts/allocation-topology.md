# Allocation topology and process-supervisor contract

## Placement classes

### `single-node-multi-gpu`

- exactly one allocated compute node and one job-scoped NFD;
- one Controller process, one User process, and three distinct Provider
  processes/identities;
- Stage 0/1/2 mapped to distinct allocated GPU UUIDs;
- local provider containers bind the same node-local NFD socket/state path;
- dependencies still use the NDNSF/NDN data path, but no cross-node claim exists.

### `multi-node`

- at least two allocated compute nodes and exactly one NFD per unique node;
- the same logical Controller/User/three-Provider role graph;
- one or more stage dependencies cross an explicit NFD face/route;
- TCP is the default selected transport; UDP is diagnostic unless the candidate
  selects UDP;
- placement change creates a new candidate/cell identity.

## Frozen process map

Each process entry binds:

```text
processId, kind, role, identityRef, nodeRank, taskRank, gpuRank/gpuUuid,
nfdSocket, commandDigest, readinessInputs, readinessOutput, shutdownOrder
```

The launcher MUST use Slurm-managed steps (`srun` or a generated multi-program
map), not background login-node processes. One node supervisor owns its NFD and
children, uses bounded readiness barriers, captures PID/task/exit records, and
terminates the process group on normal exit, TERM, INT, timeout, or partial
startup failure.

## Readiness order

1. scratch, binds, SIF, GPU mapping;
2. one NFD per unique node;
3. selected-transport faces/routes where multi-node;
4. ServiceController and identity/certificate service;
5. Provider permissions, artifact/backend readiness, and capability publish;
6. User permission readiness;
7. candidate request.

No later phase starts on partial readiness. Failure records the last satisfied
barrier and tears down all started children.

## Evidence

Retain requested and actual placement, node/task/GPU maps, NFD configs/PIDs,
faces/routes, process commands by digest, readiness timestamps, per-process exit,
selected and diagnostic transport results, and zero-survivor audits.
