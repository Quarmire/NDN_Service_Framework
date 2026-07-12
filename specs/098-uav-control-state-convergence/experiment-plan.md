# Code Experiment Plan

## Material Passport

- Origin Skill: experiment-agent
- Origin Mode: plan
- Origin Date: 2026-07-11T23:30:00-05:00
- Verification Status: UNVERIFIED
- Version Label: code_plan_v1

## Experiment Overview

- **Title**: State-gated UAV control under 5% MiniNDN loss
- **Objective**: Test whether fixed-clock advancement, rather than Takeoff
  transport loss, caused Spec 097's four `not-armed` outcomes.
- **Hypothesis**: Waiting for accepted Arm plus fresh armed telemetry will
  eliminate Takeoff `not-armed` outcomes without command retries.
- **Type**: network simulation

## Variables And Design

- **Independent variable**: fixed-clock baseline versus observed-state sequence.
- **Primary dependent variable**: Takeoff dispatch after accepted Arm.
- **Secondary dependent variables**: full control completion, convergence
  expiry, command terminal stages, lifecycle aborts, elapsed convergence time.
- **Controls**: topology, 5% loss, binaries, logging, timeout, command order,
  automatic retry disabled, one command attempt per step.
- **Confounds**: random loss realization, telemetry polling phase, process
  startup timing, cached Targeted tokens.
- **Design**: immutable Spec 097 baseline (n=5) versus one prospective treatment
  cell (n=5); no replacement repetitions.

## Setup

- **Language/Framework**: Python campaign, C++ Ground Station, MiniNDN
- **Working Directory**: `/home/tianxing/NDN/ndn-service-framework`
- **Dependencies**: current repository build plus MiniNDN
- **Environment**: passwordless sudo, setuid sudo, no concurrent MiniNDN campaign

## Inputs

| Input | Path | Description |
|---|---|---|
| Baseline | `results/spec097-uav-targeted-control-loss05-current-final` | Immutable five-run 5% cell |
| Treatment | `results/spec098-uav-control-state-loss05-current-final` | New five-run 5% cell |

## Expected Outputs

| Output | Path | Format | Success Criterion |
|---|---|---|---|
| Campaign summary | `results/spec098-uav-control-state-loss05-current-final/campaign-summary.json` | JSON | Five retained runs |
| Run table | `results/spec098-uav-control-state-loss05-current-final/campaign-runs.csv` | CSV | Terminal command and wait stages |
| Evidence | `specs/098-uav-control-state-convergence/evidence/final-validation.md` | Markdown | Counts, exact intervals, bounded claim |

## Monitoring Configuration

- **Timeout**: 180 seconds per repetition
- **Monitor files**: campaign summary and per-run Ground Station logs
- **Metric keys**: accepted, controlCompletion, stateConvergenceComplete,
  lifecycleAbort, unterminatedCommandAttempts, unterminatedAutomationWaits
- **Retry policy**: none

## Analysis Plan

- **Primary metric**: accepted Arm responses followed by Takeoff `not-armed`.
- **Functional threshold**: zero such outcomes and zero unterminated waits.
- **Comparison**: Spec 097 baseline 4/5 `not-armed` after Arm attempt versus
  treatment.
- **Uncertainty**: report Clopper-Pearson 95% intervals for completion and
  convergence proportions. Fisher's exact test may be descriptive only; n=5
  per cell does not justify a general reliability conclusion.
- **Stop rule**: run all five repetitions exactly once; preserve failures and
  do not tune or rerun.
