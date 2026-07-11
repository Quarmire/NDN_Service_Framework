# Final Structural Decision

Measured against the program baseline commit `2235fddc`, the largest Core
implementation files changed as follows:

| File | Baseline lines | Final lines | Delta | Decision |
|---|---:|---:|---:|---|
| `pythonWrapper/ndnsf/service.py` | 2,236 | 1,317 | -919 | Keep cohesive; no split now. |
| `ServiceUser.cpp` | 7,950 | 7,289 | -661 | Defer any split to a behavior-preserving Core refactor. |
| `ServiceProvider.cpp` | 9,156 | 8,628 | -528 | Defer any split to a behavior-preserving Core refactor. |
| `NDNSF-UAV-APP/ground-station/GroundStationServiceContainer.inc.hpp` | 8,460 | 8,501 | +41 | UAV-owned maintainability debt; review by 2026-09-30. |

`service.py` is now below the provisional 1,500-line review threshold and
remains a coherent user/provider facade. Splitting either large C++ runtime
translation unit would be a behavioral refactor, not an Occam deletion, and
would mix ownership movement with call-path changes. `GroundStationServiceContainer`
is still large, but its policy is correctly UAV-owned and its size does not
block the Core boundary.

No new child is opened inside Spec 084. Future splits require dedicated specs,
their own ABI/call-graph inventories, and matched performance gates. This is an
explicit deferral, not a claim that the files are ideally sized.

CodeGraph was synchronized before this decision. The final boundary query found
Repo classes under the Repo namespace, DI policy under the DI package, and no
remaining public Core coordination API after commit `f714c99`.
