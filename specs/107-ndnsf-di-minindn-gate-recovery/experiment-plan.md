# Preregistered Experiment Plan: Spec 107 Candidate C1

## Material Passport

- Origin Skill: experiment-agent
- Origin Mode: plan
- Origin Date: 2026-07-12
- Verification Status: UNVERIFIED
- Version Label: spec107_code_plan_v1

## Experiment Overview

- **Title**: Generation-scoped Qwen MiniNDN performance and live recovery
- **Objective**: Determine whether one generation-scoped collaboration session
  passes the unchanged local candidate thresholds and live recovery gates.
- **Hypothesis**: Repeated per-token collaboration setup is the dominant
  avoidable delay; removing it yields correct, bounded, recoverable service.
- **Type**: Environment-sensitive deterministic systems benchmark

## Setup

- Working directory: repository root
- Runtime: local MiniNDN, three CPU ONNX providers, one controller, one user
- Candidate namespace: `spec107-c1`
- Model/workload: exactly the frozen Spec 105 Qwen profile
- Artifact policy: one verified content-addressed ONNX set; no per-cell export
- Logging: INFO acceptance; sampled timeline only in diagnostic cells

## Inputs

| Input | Path | Description |
|---|---|---|
| Lineage | `lineage-lock.json` | Frozen Spec 105 identities |
| Model profile | `examples/ndnsf-di-qwen-pilot.model.json` | Frozen model, tokenizer, prompt, stages, and output length |
| Campaign profile | `examples/ndnsf-di-qwen-pilot.campaign.json` | Frozen topology, load, duration, logging, timeout, and thresholds |
| Fault profile | `examples/ndnsf-di-qwen-pilot-faults.campaign.json` | Frozen fault cells, triggers, outcomes, and cleanup rules |
| ONNX artifacts | `results/spec107-artifacts/<digest>/` | Read-only stage binaries |
| Diagnostic commands | `diagnostic-command-profile.json` | Exact two-cell arguments, expected tokens, and reusable artifact bindings |
| Baseline | Spec 105 matched single-node evidence | Frozen 6,854.20 ms p95 and tokens |

## Expected Outputs

| Output | Path pattern | Format | Success criterion |
|---|---|---|---|
| Attribution | `results/spec107-c1-diagnostic-*` | JSON/CSV/log | coverage/reconciliation/dominance pass |
| Performance | `results/spec107-c1-performance-r[1-3]-*` | JSON/CSV/log | every repetition passes SC-003/004 |
| Fault matrix | `results/spec107-c1-live-fault-<cell>-*` | JSON/log | all eight cells pass SC-005/006 |
| Canary/ops | `results/spec107-c1-canary-*`, `...operations-*` | JSON/log | SC-007/008 pass |
| Soak | `results/spec107-c1-soak-*` | JSON/CSV/log | SC-009 passes |
| Gate | `specs/107-.../release-gate.json` | JSON | mechanical SC-011 verdict |

## Monitoring Configuration

- Diagnostic timeout: bounded by command preregistration
- Performance window: exactly 60 measured seconds plus original request deadline
- Soak window: exactly 24 hours after warmup
- Monitor: process registry, result manifest, request outcomes, sampled timeline,
  metrics snapshots, free space, RSS, queue/wait/lease counts
- Hard stops: token/security/identity mismatch, unowned target, cleanup failure,
  disk preflight failure, malformed evidence

## Analysis Plan

- Primary metrics: exact tokens, per-repetition completion/throughput/p95 ratio,
  live-fault authoritative outcome
- Thresholds: SC-002 through SC-011; no post-hoc change
- Comparison: frozen Spec 105 workload/baseline; predecessor negative result remains separate
- Statistical treatment: descriptive p50/p95/p99 and bootstrap intervals; hard
  per-repetition thresholds remain authoritative
- Reproducibility classification: environment-sensitive for timing, deterministic
  for identities/tokens/contracts. Do not require identical timing on rerun and
  do not rerun frozen acceptance cells.

## Sample Size, Ordering, and Missing Data

- The sample size is an acceptance contract, not an inferential power claim:
  three repetitions each offer 60 generations. Each repetition is its own
  deployment decision unit and must pass independently.
- At 60 offers, the >=99% completion requirement permits zero incomplete
  generations per repetition (`59/60 < 99%`). All offered requests, including
  unfinished or harness-failed requests, stay in the denominator.
- Performance repetition IDs, commands, output roots, and order are frozen
  before repetition 1. There is no randomization or blinding because the cells
  are sequential environment-sensitive system trials; CPU/load/cache/disk facts
  are captured to expose order effects rather than pretending they are absent.
- The positive fault control precedes the eight cells. Fault order is frozen in
  the campaign profile; cleanup failure terminates the remaining sequence.
- Missing metrics, malformed evidence, a stopped host process, or a started but
  unfinished cell is failure/invalid evidence under its preregistered rule. It
  is never imputed, omitted, or replaced.

## Bias and Fallacy Controls

- No pooling, optional stopping, replacement repetition, survivor-only latency,
  or failed-cell deletion.
- Warmup outside measurement and logging identity frozen.
- All offered requests remain in denominators.
- All 11 distributed-systems fallacies are checked against live evidence.
- Diagnostic instrumentation effects are measured and excluded from acceptance.
