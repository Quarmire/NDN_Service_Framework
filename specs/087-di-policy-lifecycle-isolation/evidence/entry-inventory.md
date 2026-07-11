# Entry Inventory

- Default package currently exports advisory and semantic cache symbols.
- `default_planner_registry()` currently registers handler-less placeholders.
- `retry.py` currently infers retry safety from English error substrings.
- `merge_provider.py::DeploymentManager` retains a local ref-count authority.
- Exact Forward Cache is provider-local and must remain in the default runtime.
- CodeGraph was queried before exact text scans; a full sync currently hits the
  known `Maximum call stack size exceeded` defect on this large checkout.
