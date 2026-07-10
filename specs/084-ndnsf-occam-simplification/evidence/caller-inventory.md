# Caller Inventory Baseline

CodeGraph status on 2026-07-10 after the entry-gate commits: up to date, 2,137
files, 47,391 nodes, 161,121 edges.

Primary semantic query:

```bash
codegraph explore "Execution lease authority, coordinator fallback, DI deployment lifecycle, application-specific exports in pythonWrapper ndnsf, and tests that cover these paths"
```

Key current facts:

- `pythonWrapper/ndnsf/service.py` contains DI `ExecutionArtifact`,
  `ExecutionArtifactSpec`, materialization, deployment publication, coordinator
  lease calls, and Repo-related artifact retrieval.
- `pythonWrapper/ndnsf/coordination.py` and `pythonWrapper/ndnsf/__init__.py`
  expose coordination concepts from the generic Python surface.
- Current V2 permission checks still call `UserPermissionTable::queryPermission`;
  the table cannot be classified as wholly V1.

Exact inventory command:

```bash
python3 tools/maintenance/ndnsf_occam_audit.py . --json
```

Final Phase 1 result after excluding vendored/generated trees and the audit
script's own rule literals: 186 findings, 93 classified active. Rule totals:
V1 invocation 43, Core application leakage 114, handler-less planner 6, and
legacy-contract field 23. Historical-spec matches increased after creating 085;
they remain separately classified. These are inventory matches, not automatic
deletion decisions. Each child must rerun CodeGraph plus exact `rg` on the
specific symbols it owns.

Focused V1 command:

```bash
rg -n "PublishRequest|BloomFilter|searchByFunctionName|parsePermissionTokenName" \
  ndn-service-framework pythonWrapper NDNSF-DistributedInference \
  NDNSF-DistributedRepo NDNSF-UAV-APP tests examples
```

Focused Core-boundary command:

```bash
rg -n "ExecutionArtifactSpec|RepoDataPlaneProducer|CoordinationIntent|NotImplementedError" \
  pythonWrapper NDNSF-DistributedInference NDNSF-DistributedRepo tests examples
```
