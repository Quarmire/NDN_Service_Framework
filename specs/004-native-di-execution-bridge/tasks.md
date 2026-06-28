# Tasks: Native DI Execution Bridge

**Input**: Design documents from `specs/004-native-di-execution-bridge/`

**Tests**: Mark each task complete only after its acceptance command passes.

## Phase 1: Local Execution Baseline In Python Harness

- [x] P1 Add `--local-execution-only` to
  `Experiments/NDNSF_DI_NativeTracer_Minindn.py`

**Acceptance**:

```bash
python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --local-execution-only \
  --out /tmp/ndnsf-di-execution-bridge-local
```

The run must write `local-execution-timing.csv` and
`summary.json.localExecution.status=executed`.

**Accepted**: Implemented local execution mode with schema, manifest, and
provider-session C++ smoke commands. Evidence:
`/tmp/ndnsf-di-execution-bridge-local/local-execution-timing.csv`.

---

## Phase 2: Unified Evidence Semantics

- [x] P2 Add `localExecution` to normal evidence and keep `userExecution` and
  network `dependencyExecution` honest

**Acceptance**:

```bash
python3 - <<'PY'
import json
summary = json.load(open('/tmp/ndnsf-di-execution-bridge-local/summary.json'))
assert summary['localExecution']['status'] == 'executed'
assert summary['userExecution']['status'] == 'gated'
assert summary['dependencyExecution']['status'] in {
    'local-baseline-executed', 'gated'
}
print('NDNSF_DI_EXECUTION_BRIDGE_SUMMARY_OK')
PY
```

**Accepted**: Summary records `localExecution.status=executed` while keeping
`userExecution.status=gated` and network dependency execution explicitly gated.
Default and alternate MiniNDN bridge runs also record
`localExecution.status=executed`.

---

## Phase 3: Documentation

- [x] P3 Update project docs and feature docs with the next big NDNSF-DI plan

**Acceptance**:

```bash
python3 - <<'PY'
from pathlib import Path
for path in [
    Path('docs/native-di-roadmap.md'),
    Path('docs/experiments.md'),
    Path('docs/build-and-test.md'),
]:
    text = path.read_text()
    assert 'local execution baseline' in text.lower()
    assert 'full network execution' in text.lower()
print('NDNSF_DI_EXECUTION_BRIDGE_DOCS_OK')
PY
```

**Accepted**: Roadmap, experiment docs, and build docs now describe the local
execution baseline and the remaining full network execution gate.

---

## Phase 4: Regression Validation

- [x] P4 Run quick-smoke and focused native DI regression tests

**Acceptance**:

```bash
python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py --quick-smoke
build/unit-tests --run_test=NativeArtifactMaterializerRejectsHashMismatch,NativeProviderReadinessAckControlsSelectionEligibility,NativeProviderHandlerExtractsOnlyFinalRoleResponse,NativeExecutionPlanGeneratedJsonDrivesProviderSessionSkeleton
```

**Accepted**: Quick-smoke, local execution bridge summary check, documentation
check, default and alternate sudo MiniNDN bridge runs, focused native DI unit
tests, and full `build/unit-tests` passed.

## Dependencies & Execution Order

- P1 enables P2.
- P2 and P3 can proceed together once the summary shape is stable.
- P4 closes the gate.

## Implementation Strategy

1. Reuse the generated native tracer policy bundle.
2. Run existing C++ smoke executables instead of adding a second local runner.
3. Validate timing CSV against assignment rows.
4. Preserve the distinction between local in-memory dependency execution and
   full network dependency execution.
