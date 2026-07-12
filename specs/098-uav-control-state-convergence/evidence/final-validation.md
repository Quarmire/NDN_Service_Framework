# Final Validation Evidence

## Material Passport

- Origin Skill: experiment-agent
- Origin Mode: validate
- Origin Date: 2026-07-11T23:55:00-05:00
- Verification Status: ANALYZED
- Version Label: validation_v1

## Validation Report

- **Source**: Spec 097 canonical baseline plus Spec 098 prospective treatment
- **Overall Confidence**: CAUTION

## Implementation And Reproduction

The Ground Station auto-MAVLink worker now advances through a monotonic,
dispatch-once state object. It observes telemetry readiness before Arm, armed
telemetry before Takeoff, airborne telemetry before Land, and disarmed
telemetry before the final Emergency Stop. Convergence expiry is terminal and
never retries or bypasses a flight command.

The treatment ran once with:

```bash
python3 Experiments/NDNSF_UAV_Stream_Control_Isolation_Campaign.py \
  --out results/spec098-uav-control-state-loss05-current-final \
  --workload-modes control-only --runs 5 --loss-percent 5 \
  --auto-stop-seconds 60
```

All five repetitions are retained. `automaticRetry=false`; no command timeout,
loss setting, security path, or safety gate was changed.

## Results

| Metric | Spec 097 baseline | Spec 098 treatment | 95% exact interval |
|---|---:|---:|---:|
| Control completion | 1/5 (20%) | 3/5 (60%) | baseline 0.5%-71.6%; treatment 14.7%-94.7% |
| Full automation sequence | not instrumented | 2/5 (40%) | treatment 5.3%-85.3% |
| `not-armed` after accepted Arm | 3/4 (75%) | 0/3 (0%) | baseline 19.4%-99.4%; treatment 0%-70.8% |
| Armed telemetry before Takeoff | 1/4 accepted Arm | 3/3 accepted Arm | descriptive |
| Lifecycle aborts | 0/5 | 0/5 | descriptive |
| Duplicate command dispatch | not instrumented | 0/5 | descriptive |
| Unterminated waits/attempts | 0 | 0 | descriptive |

Fisher's exact two-sided p-values are 0.5238 for control completion and 0.1429
for the race-specific outcome. Both are non-significant and the intervals are
wide. The treatment supports a bounded mechanism claim: when Arm succeeded,
the sequencer observed armed telemetry and did not locally block Takeoff as
`not-armed`. It does not establish general reliability improvement at 5% loss.

Per-run outcomes:

- run 01-02: full sequence completed;
- run 03: initial telemetry converged, Arm timed out, Emergency Stop completed;
- run 04: initial telemetry convergence expired, Emergency Stop completed;
- run 05: Arm/Takeoff/Land completed, disarmed convergence expired before
  Emergency Stop.

The treatment therefore moves the remaining boundary earlier and later in the
state chain: initial telemetry/Arm delivery and post-Land disarmed observation.
These failures are preserved; no tuning or rerun was performed.

## Statistical Findings

| Metric | Test | Value | Effect Size | Confidence |
|---|---|---|---|---|
| Control completion | Fisher exact, two-sided | p=0.5238 | risk difference +0.40 | CAUTION |
| Race-specific `not-armed` | Fisher exact, two-sided | p=0.1429 | risk difference -0.75 | CAUTION |
| Treatment sequence completion | exact binomial | 2/5, CI 0.053-0.853 | descriptive | CAUTION |

## Warnings

| Type | Detail | Affected |
|---|---|---|
| Small sample | Five runs per cell give wide exact intervals | all proportions |
| Non-paired loss | Baseline and treatment used different stochastic loss realizations | comparison |
| Conditional denominator | Race metric includes only accepted Arm responses | race-specific result |
| Selected baseline | Spec 098 followed an unusually poor 1/5 baseline | improvement interpretation |

## Fallacy Scan

**Coverage: 11/11 checked.**

| Fallacy | Severity | Assessment |
|---|---|---|
| Simpson's paradox | NOTE | One topology/cell; no hidden subgroup reversal was tested. |
| Ecological fallacy | NOTE | Claims remain at run and mechanism level, not individual packets or hardware. |
| Berkson's paradox | CAUTION | The 5% scenario was selected after observing failures; generalization is bounded. |
| Collider bias | NOTE | No adjusted regression or conditioned control variable is used. |
| Base-rate neglect | NOTE | No classifier accuracy or prevalence claim is made. |
| Regression to the mean | CAUTION | The poor 1/5 baseline could naturally improve; mechanism evidence is separated from rate improvement. |
| Survivorship bias | NOTE | All five treatment repetitions, including failures, are retained. |
| Look-elsewhere effect | NOTE | Primary race and completion metrics were frozen before treatment. |
| Garden of forking paths | NOTE | One treatment was run without tuning; added sequence reporting was pre-specified. |
| Correlation versus causation | CAUTION | Temporal/mechanism evidence supports only the narrow sequencing claim, not general reliability causation. |
| Reverse causality | NOTE | Arm response precedes armed observation, which precedes the Takeoff decision. |

## Reproducibility

- **Method**: one prospective stochastic treatment with immutable raw logs; no
  independent rerun because the protocol forbids replacing or repeating the
  frozen cell.
- **Verdict**: CANNOT_VERIFY independently; raw artifacts and exact command are
  preserved for audit.

## Verification

- Ground Station and unit-tests built successfully.
- Full C++ unit-test module: 218/218 passed.
- UAV protocol-state suite: 40/40 passed.
- Focused Python suite: 19/19 passed.
- Ten baseline/treatment runs contain zero known lifecycle abort markers.
- Treatment has zero duplicate dispatches and zero unterminated waits.

## Residual Risk And Next Boundary

Spec 098 removes the fixed-clock `not-armed` race but does not solve lossy
control reliability. The next investigation should isolate initial telemetry
availability and Arm timeout behavior before considering any retry policy.
Post-Land disarmed convergence should be measured separately because it affects
cleanup completeness, not Takeoff dispatch.
