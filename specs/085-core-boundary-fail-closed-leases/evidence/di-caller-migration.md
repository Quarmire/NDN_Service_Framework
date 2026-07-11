# DI Caller Migration

Date: 2026-07-10

## Execution lease callers

- `examples/DI_NativeProviderExecutable.cpp` registers the secured Targeted
  `/Inference/Control/Lease` service and enables fail-closed handler validation
  only with `--require-execution-lease`.
- `native_di_tracer/user_driver.py --execution-leases` performs prepare-all,
  commit-all, assignment binding, collaboration, and finally-style release.
- Native provider roles assigned to one provider reuse one transaction
  activation idempotently. The user releases once after the whole collaboration.
- Python providers use `register_python_execution_lease_service`; both adapters
  delegate authority to the bound Core `ProviderExecutionLeaseTable`.

## Boundary moves

| Old generic surface | New owner | Repository callers |
|---|---|---|
| `ExecutionArtifact*`, `prepare_execution`, publish helper | `ndnsf_distributed_inference.artifact_deployment` | DI provider/client |
| deployment discovery/wait/provider preference | `ndnsf_distributed_inference.deployment` | DI GUI and NativeTracer user driver |
| coordinator/global-refCount acquire/release/evict | removed | none |
| error-string retry | `ndnsf_distributed_inference.retry` | NativeTracer user driver and DI scenario tests |
| `RepoDataPlaneProducer` Python adapter | `py_repoclient` | DI Repo runtime |

The serialized artifact bytes, generic encrypted large-data path, Repo packet
names, storage behavior, and segmented fetch helpers are unchanged.
