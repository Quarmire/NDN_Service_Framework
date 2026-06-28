# Tasks: Full Network NativeTracer User Driver

**Input**: Design documents from
`specs/005-full-network-native-tracer-user-driver/`

## Phase 1: Deterministic Serve Runner

- [x] P1 Add a `di-native-provider --serve` tracer deterministic runner mode

**Acceptance**:

```bash
build/examples/di-native-provider --help 2>&1 | grep tracer-deterministic-runner
```

The option must not require `--check-only` or `--wiring-check-only`.

---

## Phase 2: NativeTracer User Driver

- [x] P2 Add a Python NativeTracer user driver that submits
  `/Inference/NativeTracer` with collaboration roles and dependencies

**Acceptance**:

```bash
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/user_driver.py --dry-run
```

---

## Phase 3: MiniNDN Full-Network Harness

- [x] P3 Extend `Experiments/NDNSF_DI_NativeTracer_Minindn.py` with a
  full-network run mode

**Acceptance**:

```bash
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network \
  --out /tmp/ndnsf-di-full-network-default \
  --assignment default
```

`summary.json` must record:

```text
userExecution.status=executed
dependencyExecution.status=executed
runnerMode=deterministic-tracer
```

---

## Phase 4: Docs And Regression

- [x] P4 Update docs and run focused/full validation

**Acceptance**:

```bash
python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --quick-smoke
build/unit-tests --run_test=NativeArtifactMaterializerRejectsHashMismatch,NativeProviderReadinessAckControlsSelectionEligibility,NativeProviderHandlerExtractsOnlyFinalRoleResponse,NativeExecutionPlanGeneratedJsonDrivesProviderSessionSkeleton
build/unit-tests
```

## Dependencies

- P1 enables P3.
- P2 enables P3.
- P3 enables P4.

## Implementation Strategy

1. Reuse generated `native-execution-plan.json` and `controller.policies`.
2. Reuse `ServiceUser.request_collaboration()` instead of creating a special
   AI-specific core API.
3. Keep deterministic tracer compute explicit in evidence.
4. Promote real ONNX NativeTracer artifacts to the next feature gate.

## Completion Evidence

```bash
./waf build --targets=di-native-provider
python3 -m py_compile Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  examples/python/NDNSF-DistributedInference/native_di_tracer/user_driver.py
python3 examples/python/NDNSF-DistributedInference/native_di_tracer/user_driver.py --dry-run
sudo -n python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --full-network --core-trace \
  --out /tmp/ndnsf-di-full-network-final \
  --assignment default --provider-check-timeout 45
```

`/tmp/ndnsf-di-full-network-final/summary.txt` records:

```text
status=SUCCESS
runnerMode=deterministic-tracer
userExecution=executed
dependencyExecution=executed
```
