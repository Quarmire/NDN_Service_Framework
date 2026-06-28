# Feature Specification: Full Network NativeTracer User Driver

**Feature Branch**: `005-full-network-native-tracer-user-driver`

**Created**: 2026-06-24

**Status**: Accepted

**Input**: User request: "005: Full Network NativeTracer User Driver：实现一个真正提交 /Inference/NativeTracer 的 user driver，让 provider 从 --wiring-check-only 进入 --serve，并把 dependency exchange 从 in-memory baseline 推到 NdnsfCollaborationDependencyIo 的真实 NDNSF large-data path。这个完成后，userExecution.status 和 network dependencyExecution.status 才能改成 executed。"

## Scope

This feature executes the NativeTracer service through real NDNSF network
semantics:

```text
ServiceController
native providers in --serve mode
NativeTracer user driver
Request / ACK / Selection / Response
NdnsfCollaborationDependencyIo publish/fetch of planned activation objects
```

The current NativeTracer model files are placeholder artifacts. Therefore this
feature uses a deterministic tracer runner inside `di-native-provider --serve`
so the network and dependency path can execute now. Real ONNX model execution is
the next gate after this feature.

## User Scenarios & Testing

### User Story 1 - Submit A Real NativeTracer Request (Priority: P1)

As an NDNSF-DI developer, I need a user driver that submits
`/Inference/NativeTracer` through the NDNSF collaboration API, so the evidence
proves the user path instead of only provider wiring.

**Independent Test**: Run the user driver against a MiniNDN topology with
providers in serve mode; `summary.json.userExecution.status` is `executed`.

### User Story 2 - Exchange Dependencies Through NDNSF (Priority: P2)

As an NDNSF-DI developer, I need source/head/merge roles to exchange planned
activation objects through `NdnsfCollaborationDependencyIo`, so dependency
evidence moves beyond in-memory local execution.

**Independent Test**: Provider logs contain
`NDNSF_DI_PROVIDER_HANDLER_TIMING` for all roles, and the user receives a
successful final response.

### User Story 3 - Keep Compute Claims Honest (Priority: P3)

As an NDNSF-DI developer, I need the summary to record that this is a
deterministic tracer runner rather than real ONNX inference, so papers and docs
do not overclaim.

**Independent Test**: `summary.json.runnerMode` records
`deterministic-tracer`, and the roadmap keeps real ONNX execution as the next
gate.

## Requirements

- **FR-001**: Add a NativeTracer user driver that calls
  `ServiceUser.request_collaboration()` for `/Inference/NativeTracer`.
- **FR-002**: Add a serve-mode provider option for deterministic NativeTracer
  role runners, without using `--wiring-check-only`.
- **FR-003**: Update `NDNSF_DI_NativeTracer_Minindn.py` with a full-network
  mode that starts controller, providers in `--serve`, and the user driver.
- **FR-004**: The full-network run MUST set `userExecution.status=executed`
  only after the user driver returns a successful response.
- **FR-005**: The full-network run MUST set `dependencyExecution.status=executed`
  only after all role timing logs are observed.
- **FR-006**: The full-network summary MUST record `runnerMode`.
- **FR-007**: Docs MUST distinguish deterministic tracer network execution from
  real ONNX execution.

## Success Criteria

- **SC-001**: `python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --quick-smoke`
  passes.
- **SC-002**: Full-network default MiniNDN run passes and writes
  `userExecution.status=executed`.
- **SC-003**: Full-network default MiniNDN run writes
  `dependencyExecution.status=executed`.
- **SC-004**: Focused native DI tests and full unit tests pass.

## Acceptance Evidence

Accepted on 2026-06-24 with:

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network \
  --core-trace \
  --out /tmp/ndnsf-di-full-network-final \
  --assignment default \
  --provider-check-timeout 45
```

Observed summary:

```text
status=SUCCESS
runnerMode=deterministic-tracer
securityBootstrap=executed
userExecution=executed
dependencyExecution=executed
```
