# Implementation Plan: Real MiniNDN Native DI Tracer

**Branch**: `003-native-di-real-minindn` | **Date**: 2026-06-24 |
**Spec**: [spec.md](spec.md)

## Summary

Build the next NDNSF-DI validation layer: a real MiniNDN launcher that reuses
the native tracer policy bundle, assigns native roles to separate MiniNDN nodes,
starts MiniNDN/NFD, runs role-specific native provider checks, records security
bootstrap status, and writes evidence using the same contract shape as feature
002. Full user request execution remains explicitly gated until a native tracer
user driver and real runnable artifacts are available.

## Technical Context

**Language/Version**: Python 3 MiniNDN harness; C++17 native provider binaries

**Primary Dependencies**: MiniNDN, NFD, ndn-cxx, NDNSF dynamic runtime,
`di-native-provider`, native tracer policy generator

**Storage**: Filesystem evidence directories under `results/` or `/tmp`

**Testing**: quick-smoke, default/alternate evidence runs, hard MiniNDN gate,
focused Boost tests, full `build/unit-tests`

**Target Platform**: Ubuntu/Linux development host with passwordless sudo

**Project Type**: Mixed C++ examples plus Python experiment harnesses

**Performance Goals**: Correct topology/evidence capture, not throughput
optimization

**Constraints**: MiniNDN-first validation; preserve NDNSF security assumptions;
do not reintroduce generated service/stub APIs; do not claim full inference
with placeholder artifacts

**Scale/Scope**: One native tracer service with `/Backbone`,
`/Head/Shard/0`, `/Head/Shard/1`, and `/Merge` mapped to four MiniNDN nodes

## Constitution Check

- **Canonical Dynamic Runtime**: Pass. Uses current native DI runtime and
  `di-native-provider`; no generated stubs.
- **Security Is Part Of The Data Path**: Pass. Records security bootstrap
  status and keeps full execution gated if security/user path is not executed.
- **CodeGraph First, Source Verified**: Pass. Native provider and MiniNDN
  harness entry points were inspected before design.
- **Spec-Driven Changes**: Pass. This feature has Spec Kit artifacts before
  implementation.
- **Verify With The Right Scope**: Pass. MiniNDN is the primary validation
  surface; local tests remain supporting regressions.

## Project Structure

```text
specs/003-native-di-real-minindn/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   ├── evidence.md
│   └── assignments.md
├── checklists/
│   └── requirements.md
└── tasks.md

Experiments/
└── NDNSF_DI_NativeTracer_Minindn.py
```

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| Full user request path gated | Current tracer artifacts are placeholders and no dedicated native tracer user driver is available | Claiming full execution would make the evidence misleading |
