# Final Validation

## Material Passport
- Origin Skill: experiment-agent
- Origin Mode: validate
- Origin Date: 2026-07-12
- Verification Status: ANALYZED

Frozen result path: `results/spec101-uav-telemetry-observation-lease-loss05-final`.
The five-run 5% cell ran once.

| Metric | Spec 100 | Spec 101 |
|---|---:|---:|
| Control/full sequence | 2/5 | 3/5 |
| Armed wait satisfied | 3/4 | 3/4 |
| Ground telemetry invisible | 1 | 1 |
| Initial telemetry timeout/no Arm | 1 | 1 |
| Lifecycle abort/duplicate/unterminated | 0 | 0 |

The visibility failure received two sequential telemetry attempts, timing out
at 5002 and 5005 ms. Thus the lease worked, but two opportunities did not repair
that stochastic run. Three successful armed waits used response latencies of
39-294 ms. Completion movement is descriptive only (Fisher two-sided p=1.0);
no reliability improvement claim is supported.

Verification: Python 24/24; C++ 219/219 and 16264 assertions; Waf build and
strict Spec Kit audit pass. Fallacy scan 11/11: small selected stochastic cells,
regression to mean, and causal overreach require caution; all failures are
retained and one frozen analysis limits survivorship/look-elsewhere/forking risks.

Next boundary: two consecutive telemetry sender timeouts despite confirmed
drone armed state. Packet direction remains unknown.
