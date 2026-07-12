# Code Experiment Plan

## Material Passport

- Origin Skill: experiment-agent
- Origin Mode: plan
- Origin Date: 2026-07-11T23:59:00-05:00
- Verification Status: UNVERIFIED
- Version Label: code_plan_v1

## Experiment Overview

- **Title**: 5% Initial UAV Control Attribution
- **Objective**: classify telemetry/Arm and automation outcomes without policy tuning
- **Hypothesis**: corrected timestamps eliminate observer mismatches; remaining
  failures terminate in explicit sender-side, convergence, or named-unknown categories
- **Type**: networked systems diagnostic

## Setup

- **Language/Framework**: C++17, Python 3.8+, MiniNDN
- **Entry Command**: frozen command in `quickstart.md`
- **Working Directory**: `/home/tianxing/NDN/ndn-service-framework`
- **Dependencies**: existing NDNSF/MiniNDN/PX4 SITL environment
- **Environment**: existing Memphis/UCLA campaign topology

## Inputs And Outputs

| Item | Path | Criterion |
|---|---|---|
| Baseline | `results/spec098-uav-control-state-loss05-current-final` | immutable five runs |
| Treatment | `results/spec099-uav-initial-control-attribution-loss05-final` | five retained runs |
| Summary | treatment `campaign-summary.json` | attribution per run |
| Evidence | `evidence/final-validation.md` | bounded interpretation |

## Monitoring Configuration

- **Timeout**: campaign-owned 60-second measurement per run plus setup/cleanup
- **Monitor files**: launcher, Ground Station, drone, controller logs
- **Metric file/key**: `campaign-summary.json` / attribution completeness

## Analysis Plan

- **Primary metric**: 5/5 terminal or named-unknown attribution
- **Success threshold**: zero observer mismatch after fix
- **Secondary**: completion, lifecycle abort, duplicate/unterminated states
- **Comparison**: Spec 098, with completion descriptive only
- **Stop rule**: no rerun or tuning after frozen cell
