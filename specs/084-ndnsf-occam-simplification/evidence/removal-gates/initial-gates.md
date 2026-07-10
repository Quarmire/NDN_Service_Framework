# Initial Removal And Retention Gates

Each row is one initial mechanism record. All are BLOCKED until the owning child
expands the row into the full `contracts/removal-gate.md` template with exact
callers, tests, evidence, migration commits, and rollback command. `keep` rows
are also blocked until the baseline proves the retained surface is intentional.

| Mechanism | Current owner | Target/disposition | Change class | Child | Initial decision |
|---|---|---|---|---|---|
| V2 normal/Targeted invocation | Core | Core / keep | public API + wire + security | 085/086 | BLOCKED |
| V1 split names and Bloom filter | Core | none / remove | public API + ABI + wire + security | 086 | BLOCKED |
| Legacy ACK/local handler overloads | Core | none or packaged adapter | public ABI | 086 | BLOCKED |
| DI deployment lifecycle methods | Python Core wrapper | DI / move | distributed authority | 085/087 | BLOCKED |
| ExecutionArtifactSpec/materializer | Python Core wrapper | DI / move | API + large-data/security | 085 | BLOCKED |
| RepoDataPlaneProducer | Python Core wrapper/binding | Repo / move | public API + wire | 085/088 | BLOCKED |
| Retry by error string | Python Core wrapper | DI / replace with explicit idempotency | behavior + distributed retry | 085/087 | BLOCKED |
| Coordination envelope/service | Core | DI experimental or remove | public API + wire | 087 | BLOCKED |
| Provider admission/generic lease envelope | Core | Core / keep | distributed authority + security | 085 | BLOCKED |
| DI prepare/commit/abort and fragment residency | DI | DI / keep | distributed authority | 085/087 | BLOCKED |
| Semantic service cache | DI public runtime | DI experimental | behavior + output correctness | 087 | BLOCKED |
| Exact Forward Cache | DI provider | DI / keep | hot path + correctness | 087 | BLOCKED |
| Planner placeholders without handlers | DI | none / remove | public behavior | 087 | BLOCKED |
| Repo C++ and Python storage engines | Repo and DI package | Repo / ADR selects one | stored data + wire + hot path | 088 | BLOCKED |
| STORE raw-payload convenience | Repo wire API | Repo client adapter | public API + wire | 088 | BLOCKED |
| Packet batch/pull/finalize operations | Repo public surface | private authorized protocol | public/internal security boundary | 088 | BLOCKED |
| Typed plus flat legacy ACK/status | Core/DI/Repo/UAV | typed contracts; retain domain state | schema + mixed version + stored data | 090 | BLOCKED |
| C++/Python/UAV stream state | Core/UAV | Core C++ generic state | wire + binding + hot path | 089 | BLOCKED |
| Operator authority lease | UAV | UAV / keep | safety + distributed authority | 089 boundary proof | BLOCKED |

Common pending fields for every row: repository callers, external callers/ABI,
replacement or keep rationale, security invariants, persistence/wire impact,
migration commits, focused/module/MiniNDN commands, matched performance result,
independent reviewer, and rollback command.
