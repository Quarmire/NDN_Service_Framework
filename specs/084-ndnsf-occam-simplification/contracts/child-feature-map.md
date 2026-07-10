# Child Feature Contract

Spec 084 is the governance umbrella for the simplification program. It does
not authorize cross-project production edits directly. Each implementation
wave MUST be specified, audited, implemented, and accepted in its own child
feature.

| Child | Scope | Entry gate | Exit evidence |
|---|---|---|---|
| 085 Core boundary and fail-closed leases | Remove application policy from Core; repair unsafe lease fallback | DI lease-authority contract approved | Core/security tests and authority-loss regression |
| 086 V2-only invocation and permission table | Remove V1 naming, Bloom filter, deprecated token indexing, verified-dead callbacks | External ABI decision and symbol-level caller inventory | Full build, security regressions, normal/Targeted MiniNDN |
| 087 DI policy and lifecycle ownership | DI deployment records, provider leases, planner registry, experimental coordination/cache isolation | 085 complete | Coordinator-off multi-user and Qwen/NativeTracer evidence |
| 088 Repo canonical runtime and contract | Select wire contract and runtime, then migrate by parity slices | Repo ADR approved; persistence fixtures frozen | Exact packet, restart, HA, repair, catalog and performance gates |
| 089 Core stream parity and UAV migration | C++ stream state, Python binding, UAV generic-state migration | Stream parity contract and frozen UAV behavior tests | Unit parity and matched UAV MiniNDN campaign |
| 090 Typed-envelope migration | Versioned typed contracts and bounded mixed-version epoch | Field inventory classifies aliases versus domain state | Mixed-version then typed-only evidence, zero legacy use |

Child numbers are reservations. Creation MUST use the repository Spec Kit
workflow and MUST NOT overwrite an existing feature. A child may be split
again when its implementation plan still contains an unresolved architecture
choice or cannot be independently rolled back.

Dependency order:

```text
085 -> 086
085 -> 087
088 may start after the program baseline and Repo ADR
089 may start after the program baseline and stream contract
086 + 087 + 088 + 089 -> 090
all children -> 084 final acceptance
```
