# Final Validation

## Material Passport
- Origin Skill: experiment-agent
- Origin Mode: validate
- Origin Date: 2026-07-12
- Verification Status: ANALYZED

One frozen five-run 5% cell ran at
`results/spec102-uav-telemetry-provider-attribution-loss05-final`.

| Metric | Spec 101 | Spec 102 |
|---|---:|---:|
| Control completion | 3/5 | 5/5 |
| Full automation sequence | 3/5 | 4/5 |
| Armed visibility satisfied | 3/4 | 5/5 |
| Lifecycle abort/duplicate/unterminated | 0 | 0 |

Five unique telemetry sender timeouts occurred across the treatment. Three have
matching `handler-return` evidence, so the provider computed a response but the
Ground Station did not receive it before timeout. Two have no handler event, so
they failed before handler visibility (network arrival or pre-handler rejection
remain indistinguishable). The evidence therefore rejects a single-direction
failure explanation.

One run timed out on final Emergency Stop, so 5/5 control completion must not be
conflated with 5/5 full sequence. Completion change from 3/5 to 5/5 is not
significant (Fisher two-sided p=0.4444) and supports no reliability claim.

Verification: Python 25/25; C++ 219/219 with 16264 assertions; Drone/Ground
Station Waf build and strict audit pass. Fallacy scan 11/11: small selected
stochastic cells, regression to mean, and causal overreach require caution; all
runs/timeouts are retained, limiting survivorship/look-elsewhere/forking bias.

Next boundary: distinguish pre-handler network arrival from authorization/token
rejection, and distinguish handler return from response publication/forwarding.
