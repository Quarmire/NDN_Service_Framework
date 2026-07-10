# Core Boundary Migration Map

| Current symbol | Current path | Target | Disposition |
|---|---|---|---|
| `ExecutionArtifact` | `pythonWrapper/ndnsf/service.py` | DI `artifact_deployment.py` | move |
| `ExecutionArtifactSpec` | same | DI `artifact_deployment.py` | move |
| `ExecutionContext` | same | DI `artifact_deployment.py` | move |
| deployment publish/get/evict/wait | `pythonWrapper/ndnsf/service.py` | DI `deployment.py` | replace/move |
| acquire/release execution lease | same | DI `deployment.py` over provider service | replace |
| coordinator/global refCount authority | Core wrapper + DI merge provider | none | remove as correctness path |
| `RepoDataPlaneProducer` | `pythonWrapper/ndnsf/service.py` | `py_repoclient` | move/adapter |
| retry inferred from error text | Core wrapper | DI explicit idempotent policy | remove/replace |
| generic segmented/exact Data helpers | Core wrapper | Core | keep |
| generic status/runtime/network/admission envelopes | Core telemetry | Core | keep |
| coordination envelopes | Core | unchanged until Spec 087 | defer |

No old export is deleted until all repository callers use the target import and
the external ABI decision is recorded.
