# DI Lease Baseline

The current code does not implement the intended fail-closed authority model.

Current authority path in `pythonWrapper/ndnsf/service.py`:

- `_publish_deployment_ndnsd()` documents the Merge Provider/coordinator as the
  sole deployment publication authority and suppresses every exception.
- `evict_deployment()` trusts a deployment-level `refCount`.
- `acquire_execution_lease()` asks the coordination service and, on any
  exception/no suggestion, imports `ExecutionLease` from
  `ndnsf.runtime_telemetry` and returns `GRANTED_LOCAL`.
- `ndnsf.runtime_telemetry` defines no `ExecutionLease`; the fallback can raise
  `ImportError` rather than grant or reject a lease.
- `runtime_v1.py` contains `DeploymentLeaseTable` annotations and operations
  referring to `ExecutionLease`, but no repository definition was found by:

```bash
rg -n "class ExecutionLease|ExecutionLease\s*=" . \
  --glob '!third_party/**' --glob '!.codegraph/**'
```

Required 085 baseline regression: coordinator unavailable must currently
demonstrate the import/error path, then treatment must return a typed
`UNAVAILABLE`/`REJECTED` result and create no lease. This evidence does not
claim the current fallback works.
