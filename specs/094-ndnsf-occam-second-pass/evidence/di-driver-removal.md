# DI Driver And Sweep Removal

Removed:

- public `process-pool` choices/defaults from the user driver, MiniNDN harness,
  campaign wrapper, runtime profile resolver, and GUI;
- process-pool schedule helpers, worker-batch private protocol, subprocess pool,
  and dedicated tests;
- `Experiments/NDNSF_DI_RuntimeAware_RpsSweep.py` and its success-only tests.

Canonical behavior:

- `threaded` is the measured open-loop default;
- `child` remains for process-isolation diagnostics;
- strict rate work uses the NativeTracer MiniNDN harness and Spec 093 gates.

Verification:

```text
runtime-aware campaign: 27 passed
runtime-profile campaign: 1 passed
Tk/headless GUI: 20 passed
active symbol scan: no process-pool, worker batch, or removed sweep references
Python compilation: passed for all edited launchers
```
