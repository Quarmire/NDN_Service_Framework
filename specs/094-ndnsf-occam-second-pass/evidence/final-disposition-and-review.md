# Final Disposition And Adversarial Review

## Disposition

### Remove

- DI `process-pool` mode and its private worker-batch protocol: duplicate,
  invalid evidence path; threaded is canonical and child remains diagnostic.
- Deterministic runtime-aware RPS sweep: it did not exercise real scheduling,
  dependency transfer, or trace validity.
- Legacy DI runtime GUI profiles and duplicate script-role tabs: the version-2
  three-role profile and direct USER/PROVIDER/CONTROLLER tabs cover the same
  operator workflow.
- DI `repo_manifests` alias: zero callers; `artifact_references` is canonical.
- Public Repo memory-only authority and default constructors: contradicted the
  SQLite-authoritative persistence contract. A bounded memory hot cache remains.
- Ignored Repo `producer_retention_s` and private `isolated_runtime` options.
- Redundant `legacyStatus` metadata and per-object route-propagation wait.

### Consolidate

- Repo Data retrieval uses one stable per-provider `REPO-SERVING` locator plus
  the versioned Data name returned by `FETCH_PREPARE`; dynamic object names are
  not individually advertised through NLSR.
- UAV local recording now uses the same explicit SQLite-authoritative tiered
  Repo contract as other production callers.
- Current DI GUI roles share `ThreeRoleGuiProfile`; supporting project, policy,
  certificate, model, Qwen, and regression tools remain distinct tools rather
  than duplicate role launchers.

### Keep

- Core V2 invocation, NAC-ABE permission routing, one-time request/provider
  tokens, typed envelopes, fail-closed leases, and mixed ACK reader security.
- Core stream contracts for continuous publication and SegmentFetcher for named
  finite objects.
- UAV H264/FEC/ROI and mission policy in the UAV application.
- Repo SQLite persistence, bounded hot cache, exact packet identity, catalog,
  replication, quorum receipts, repair, and failure-domain placement.
- DI planner/runtime, provider inventory, fragment residency, exact/semantic
  local caches, long-context lifecycle, dependency data path, and user-side
  planning/advisory coordination.

### Defer

- Mixed ACK reader deletion remains gated by the existing next-major-release or
  2026-12-31 migration deadline.
- Broader translation-unit refactoring has no current duplicate-mechanism
  benefit and requires a separate caller/evidence gate.

## Adversarial post-implementation review

Verdict: **PASS**.

- Necessity: each removed surface had zero maintained callers or duplicated a
  tested canonical path. The implementation did not remove mechanisms solely
  because they were large.
- Boundaries: framework security and generic coordination stayed in Core;
  application policy stayed in UAV/Repo/DI. The convergence migration fixed an
  app caller rather than restoring a Core memory-only shortcut.
- Persistence and rollback: all production Repo construction is explicit and
  SQLite-authoritative. Temporary test stores use temporary SQLite or local
  fakes. Reverting each focused commit remains the rollback mechanism.
- Security: permission encryption, NAC-ABE routing, token/replay checks,
  targeted bootstrap, fail-closed leases, and typed ACK writers were untouched.
- Distributed failure behavior: the Repo network check initially failed twice
  and was not waived. The defect was traced to discarded locator metadata,
  covered by two focused regressions, and then proved on MiniNDN.
- Evidence quality: checked task boxes are supported by focused tests, full
  Python/C++ runs, symbol scans, a 60-second DI network run, and Core/Repo/UAV
  quick checks. The UAV quick check is launcher/config coverage, not a full
  flight or video campaign.
- Residual risk: the DI run is a one-rate acceptance run, not a throughput
  campaign; mixed ACK compatibility still carries bounded maintenance cost;
  full UAV video/FEC/mission network campaigns remain outside Spec 094.

