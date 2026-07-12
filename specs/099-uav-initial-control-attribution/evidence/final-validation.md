# Final Validation Evidence

## Material Passport

- Origin Skill: experiment-agent
- Origin Mode: validate
- Origin Date: 2026-07-12T00:30:00-05:00
- Verification Status: ANALYZED
- Version Label: validation_v1

## Validation Report

**Overall confidence: CAUTION.** Spec 099 corrects command-state evidence and
achieves complete sender-side attribution, but the frozen cell is a negative
control-reliability result and does not localize packet direction.

## Implementation And Execution

`FlightCommandState::makePending` now records zero RTT and attempt update time;
`makeTimeout` records terminal update time and elapsed RTT. Both asynchronous
and synchronous Ground Station command paths use the factories. Existing stale
state rejection, timeout values, telemetry polling, safety, security, and
single-attempt flight command behavior are unchanged.

The frozen treatment ran exactly once:

```bash
python3 Experiments/NDNSF_UAV_Stream_Control_Isolation_Campaign.py \
  --out results/spec099-uav-initial-control-attribution-loss05-final \
  --workload-modes control-only --runs 5 --loss-percent 5 \
  --auto-stop-seconds 60
```

All five raw repetitions are retained. Subsequent `--reparse-existing` calls
changed summaries only and did not relaunch experiments.

## Results

| Metric | Spec 098 baseline | Spec 099 treatment | 95% exact interval |
|---|---:|---:|---:|
| Control completion | 3/5 | 2/5 | treatment 5.3%-85.3% |
| Full automation sequence | 2/5 | 2/5 | treatment 5.3%-85.3% |
| Attribution complete | 5/5 after reparse | 5/5 | descriptive |
| Command observer mismatch | 1/5 | 0/5 | treatment 0%-52.2% |
| Initial telemetry sender timeout | 1/5 | 1/5 | treatment 0.5%-71.6% |
| Armed convergence expiry after Arm response | not separately reported | 2/4 | 6.8%-93.2% |
| Lifecycle abort | 0/5 | 0/5 | descriptive |
| Duplicate/unterminated automation or commands | 0 | 0 | descriptive |

Treatment boundaries are two `arm-response` completions, two
`armed-convergence-expired` failures, and one `telemetry-sender-timeout` failure.
One telemetry-timeout run launched a later Targeted observation that had no
terminal phase before shutdown; its request ID is explicitly preserved in
`unknownReasons`, while the earlier timeout still supports the boundary.

All four treatment Arm pending states logged `rtt_ms=0`, correcting the visible
epoch-as-RTT defect. No Arm timeout occurred in this stochastic cell, so the
timeout factory is unit-tested but not executed by this treatment.

Fisher's exact two-sided p-value is 1.0 for both control completion (3/5 versus
2/5) and observer mismatch (1/5 versus 0/5). The sample is too small for an
improvement claim. The measured new boundary is telemetry convergence after an
accepted Arm, not command response bookkeeping.

## Fallacy Scan

**Coverage: 11/11 checked.**

| Fallacy | Assessment |
|---|---|
| Simpson's paradox | NOTE: one topology/cell; no subgroup claim. |
| Ecological fallacy | NOTE: claims remain at run/sender-observation level. |
| Berkson's paradox | CAUTION: 5% loss was selected after prior failures. |
| Collider bias | NOTE: no adjusted model or control conditioning. |
| Base-rate neglect | NOTE: no classifier metric. |
| Regression to the mean | CAUTION: tiny stochastic cells can move naturally. |
| Survivorship bias | NOTE: all five runs and failures retained. |
| Look-elsewhere effect | NOTE: attribution completeness was predeclared primary. |
| Garden of forking paths | NOTE: one frozen run; parser refinements used same raw data. |
| Correlation versus causation | CAUTION: sender timing does not prove packet-loss direction. |
| Reverse causality | NOTE: phase timestamps establish only local temporal order. |

## Verification

- Waf Ground Station/unit-test build: success.
- Full C++ module: 219/219 tests, 16264/16264 assertions.
- UAV protocol-state suite: 41/41 tests, 646/646 assertions.
- Focused Python parser suite: 22/22 tests.
- Strict Spec Kit structure audit: PASS.
- Direct treatment scan: zero known lifecycle abort markers.

## Continuity Incident

A model-capacity notice arrived while the Waf build was active. Following the
repository continuity rule, the same build session was resumed from its verified
checkpoint and completed successfully. No experiment had started, no completed
work was restarted, and the frozen MiniNDN cell was still executed only once.

## Bounded Conclusion And Next Boundary

Implemented: correct pending/timeout state factories and attribution parsing.
Executed: pending state on four treatment Arm attempts; timeout behavior in unit
tests. Measured: 5/5 attributable runs, zero observer mismatch, but only 2/5
control completion. The next treatment should address post-Arm armed-telemetry
convergence ownership. It must not infer request/response loss or introduce
command retry without a separately specified experiment.
