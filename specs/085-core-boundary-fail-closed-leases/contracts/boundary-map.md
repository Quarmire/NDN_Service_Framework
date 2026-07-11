# Core Boundary Migration Map

| Current symbol | Current path | Target | Disposition |
|---|---|---|---|
| `ExecutionArtifact` | DI `artifact_deployment.py` | DI | moved |
| `ExecutionArtifactSpec` | DI `artifact_deployment.py` | DI | moved |
| `ExecutionContext` | DI `artifact_deployment.py` | DI | moved |
| deployment discovery/wait/preference | DI `deployment.py` | DI | moved |
| acquire/release execution lease | DI `deployment.py` over provider service | DI | replaced |
| coordinator/global refCount authority | none | none | removed as correctness path |
| `RepoDataPlaneProducer` | `py_repoclient` | Repo | moved adapter |
| retry inferred from error text | DI `retry.py` | DI | moved; bounded to DI caller |
| generic segmented/exact Data helpers | Core wrapper | Core | keep |
| generic status/runtime/network/admission envelopes | Core telemetry | Core | keep |
| coordination envelopes | Core | unchanged until Spec 087 | defer |

The old exports are deleted under the pre-1.0 breaking API decision recorded in
`evidence/python-api-decision.md`.
