# Implementation Plan: DI Rate-Aware Cost Model

**Branch**: current worktree | **Date**: 2026-06-28 |
**Spec**: [spec.md](spec.md)

## Summary

Extend the NativeTracer planner evidence with an optional target request rate.
When the rate is zero, existing behavior remains unchanged. When the rate is
positive, each candidate reports provider utilization, provider capacity queue
pressure, dependency byte rate, and dependency link utilization; these fields
are then propagated into layout and auto campaign summaries.

## Technical Context

**Language/Version**: Python 3.8 experiment and planning helpers

**Primary Dependencies**: MiniNDN harness, NativeTracer policy generator,
existing Python standard library JSON/CSV tooling

**Storage**: Filesystem evidence directories under `/tmp` or `results/`

**Testing**: Python syntax checks, planner-only runs, focused DI unit tests, and
optional MiniNDN smoke

**Target Platform**: Ubuntu/Linux development host with passwordless sudo

**Project Type**: Python experiment harness plus C++ NDNSF-DI runtime

**Performance Goals**: Preserve existing auto recommendation boundary by
default; expose rate-aware pressure fields for high-concurrency campaigns

**Constraints**: Do not change proposal slides; do not change model artifacts;
keep full-network validation MiniNDN-first

## Constitution Check

- **Canonical Dynamic Runtime**: Pass. This feature changes planner evidence and
  experiment wrappers only.
- **Security Is Part Of The Data Path**: Pass. No authorization or token path is
  changed.
- **CodeGraph First, Source Verified**: Pass. Planner and harness entry points
  are inspected before edits.
- **Spec-Driven Changes**: Pass. This feature has a dedicated spec/plan/tasks.
- **Verify With The Right Scope**: Pass. Planner-only validation is the first
  gate; MiniNDN remains the final network validation surface.

## Project Structure

```text
specs/025-di-rate-aware-cost-model/
├── spec.md
├── plan.md
└── tasks.md

examples/python/NDNSF-DistributedInference/native_di_tracer/
├── optimize_native_tracer_plan.py
├── plan_tracer.py
├── run_layout_campaign.py
└── run_auto_assignment_campaign.py

Experiments/
└── NDNSF_DI_NativeTracer_Minindn.py
```

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | N/A |
