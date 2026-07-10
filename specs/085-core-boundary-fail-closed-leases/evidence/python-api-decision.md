# Python API Compatibility Decision

**Decision**: use an explicit `0.2.0` breaking migration; do not retain a
duplicate compatibility implementation in generic `ndnsf`.

## Scope

The following application-owned symbols currently leak through the generic
`ndnsf` Python package:

- `ExecutionArtifact`, `ExecutionArtifactSpec`, and `ExecutionContext`;
- artifact deployment/materialization methods on generic service wrappers;
- `RepoDataPlaneProducer`;
- application retry decisions based on error-message strings.

## Evidence

- The C++ and Python package version is `0.1.0`.
- CodeGraph and exact caller scans show execution-artifact callers in
  `NDNSF-DistributedInference`; Repo producer callers are in the Repo runtime
  and its DI-hosted implementation.
- No repository caller requires these names to remain owned by generic Core.
- External callers cannot be proven absent, so the move must be announced as a
  minor pre-1.0 breaking release rather than silently changing `0.1.0`.

## Migration Contract

1. T009 first locks the target imports and forbidden generic exports.
2. T024-T028 add the DI- and Repo-owned APIs and migrate every repository caller
   while preserving serialized bytes, hashes, packet names, and runtime behavior.
3. The Python package and C++ library version move to `0.2.0` in the export
   deletion commit.
4. T029 removes the old generic exports and methods in the same release boundary.
5. Release notes and migration examples map each old import to its new owner.

## Compatibility Adapter Decision

No temporary runtime adapter will be added. An adapter would keep application
policy visible from generic Core, preserve the ownership ambiguity that 085 is
meant to remove, and require another removal cycle. Source migration guidance is
the compatibility mechanism.

If an independently distributed downstream package is discovered before T029,
T029 must pause. The only permitted exception is a warning-only import shim
owned by Spec 085, expiring no later than `0.3.0`; it must contain no duplicate
algorithm or state and must have a tested removal date.
