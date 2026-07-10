# Repo Canonical Runtime Decision Gate

Spec 084 requires one Repo contract and one authoritative runtime but does not
preselect C++ or Python. Child feature 088 must first produce an ADR using
frozen black-box fixtures and the criteria below.

## Mandatory Criteria

1. Exact signed Data names and wire bytes are preserved without reconstruction.
2. SQLite schema, restart, upgrade, tombstone, catalog, quorum, and repair state
   have a tested migration path.
3. The public contract and internal replication contract have stable names,
   versions, authorization policy, and malformed-input behavior.
4. The selected runtime supports bounded concurrency, cancellation, backpressure,
   observability, crash recovery, and deterministic test fixtures.
5. Python remains acceptable as an orchestration/client adapter; it must not
   retain a second authoritative storage/catalog/repair implementation.
6. Migration can proceed in independently revertible parity slices.

## Evidence

The decision uses semantic parity, security, operational complexity,
maintainability, persistence compatibility, and matched performance. A single
control-plane timing comparison cannot select the architecture. Performance is
one input, not the sole authority.

Until the ADR is approved, tasks may freeze fixtures and measure both paths but
must not delete either authoritative candidate or change stored data.

## Public And Internal Security Boundary

Public object operations and internal replication/repair operations must use
distinct service names or an equally enforceable typed operation boundary.
Internal operations require provider/node authorization and must not be
advertised as general client services. Negative tests must prove an ordinary
Repo client cannot invoke capacity reservation, quorum finalization, catalog
merge, anti-entropy, repair claim, or replica mutation operations.
