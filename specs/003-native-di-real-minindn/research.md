# Research: Real MiniNDN Native DI Tracer

## Decision: Reuse Existing MiniNDN Patterns

Use the existing DI MiniNDN harness style from `Experiments/NDNSF_DI_Yolo2x2_Minindn.py`
and `Experiments/NDNSF_DI_PyTorch2x2_Minindn.py`: `Minindn`, `AppManager`,
`Nfd`, `NdnRoutingHelper`, explicit node mappings, and per-process logs.

**Rationale**: These scripts already encode the project's MiniNDN setup,
security bootstrap style, routing setup, environment handling, and log capture.

**Alternatives Considered**: A new minimal Mininet script from scratch. Rejected
because it would duplicate fragile setup details and drift from the existing
experiment style.

## Decision: Keep Provider Execution Role-Specific

Start/check each native provider with only the role assigned to that node.

**Rationale**: The point of this feature is to prove provider-role assignment
and MiniNDN placement are explicit. A single provider running all roles would
not validate multi-provider cooperation semantics.

**Alternatives Considered**: Run one all-role provider. Rejected because it
would collapse the DI topology.

## Decision: Gate Full Request Execution

Record full user/dependency execution as gated until a native tracer user
driver and runnable artifacts are present.

**Rationale**: The native tracer currently uses placeholder artifacts for policy
and runtime wiring. A real request path needs a user driver and artifacts that
can execute under the native ONNX backend.

**Alternatives Considered**: Treat provider check-only as full execution.
Rejected because it would overstate the result.

## Decision: Add Explicit Wiring-Check Mode

Use `di-native-provider --check-only --wiring-check-only` for MiniNDN provider
placement checks.

**Rationale**: The native tracer artifacts are placeholders. Plain
`--check-only` correctly reaches the ONNX artifact loader and fails because the
placeholders are not valid ONNX models. The new flag keeps the test honest by
checking native plan/manifest/provider-role wiring without claiming model
execution.

**Alternatives Considered**: Generate fake ONNX files or mark the ONNX failure
as success. Rejected because fake artifacts would hide the real next gate, and
accepting ONNX parse failure as success would weaken the provider check.
