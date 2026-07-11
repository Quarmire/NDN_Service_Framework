# Final Removal And Retention Gates

Every program mechanism has a final disposition. `READY` means its caller,
security, regression, network, and rollback evidence is recorded by the named
child. `DEFERRED` means the mechanism remains intentionally, with an owner and
review condition.

| Mechanism | Final disposition | Gate | Evidence / rollback |
|---|---|---|---|
| V2 normal and Targeted invocation | Core, keep | READY | 086 Core/security/MiniNDN; revert `b3acfd1` |
| V1 split names and Bloom filters | remove | READY | 086 zero-symbol/build scan; revert `b3acfd1` |
| Legacy ACK/local overloads | remove verified-dead overloads; retain explicit packaged adapters | READY | 086 structural audit and full build |
| DI deployment lifecycle | DI-owned manager | READY | 085/087; revert `3918c98` or `00e4709` |
| Execution artifact materializer | DI-owned | READY | 085 byte/hash/path tests; revert `3918c98` |
| Repo data-plane producer | Repo public adapter; internal native binding retained | DEFERRED | Public Core export is gone. Internal binding backs exact packet production; Repo owns review by 2026-12-31. |
| Retry by error string | remove | READY | 085/087 explicit-idempotency tests |
| Core coordination envelope | remove | READY | `f714c99`; detached revert restored 6/6 tests |
| Provider admission/execution lease | Core, keep | READY | 085 fail-closed table, restart/concurrency MiniNDN |
| DI prepare/commit/abort and residency | DI, keep | READY | 085/087 transaction and restart tests |
| Semantic service cache | DI experimental, opt-in | DEFERRED | 087 boundary tests; DI owner; review on promotion request |
| Exact Forward Cache | DI provider-local, keep | READY | 087 exact-key and provider-local tests |
| Handler-less planner registrations | remove | READY | 087 registry test; defensive abstract guard remains |
| Repo C++ and Python network runtimes | Python adapter is sole network runtime; C++ object/local contract remains | READY | 088 ADR, parity, 3x60-second MiniNDN; revert `5aca321` |
| STORE raw-payload convenience | Repo client adapter | READY | 088 exact-name packet fixtures |
| Repo batch/pull/finalize | private authenticated operations | READY | 088 ordinary-client negative tests |
| Flat capability/status aliases | typed root authoritative; bounded mixed reader only | DEFERRED | 090; reader expires next major release or 2026-12-31; revert `72dc052` |
| C++/Python/UAV stream state | C++ Core canonical; UAV policy retained | READY | 089 parity/loss campaign; revert `01466f5` |
| UAV operator authority lease | UAV, keep | READY | 089 authority and mission regressions |

Final scans found no active V1 invocation symbol or dead build target. Broad
string scans that flag DI artifacts, Repo classes, abstract GUI methods,
optional ACK handlers, or domain `status` fields are false positives: these are
owned application behavior or defensive interfaces, not duplicate Core paths.

