# Implementation Plan: Native DI MiniNDN Tracer

**Branch**: `002-native-di-minindn-tracer` | **Date**: 2026-06-24 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/002-native-di-minindn-tracer/spec.md`

## Summary

Promote the native DI tracer from local C++ smoke evidence into a MiniNDN-ready tracer with harder data-path checks, research-grade evidence, explicit provider assignments, and an LLM planner gate.

## Technical Context

**Language/Version**: Python 3 experiment harnesses; C++17 native DI runtime and smoke binaries

**Primary Dependencies**: NDNSF dynamic runtime, ndn-cxx, NFD, ndn-svs, MiniNDN/Mininet, existing native DI tracer policy and C++ examples

**Storage**: Filesystem result directories under `results/` or `/tmp`, with logs, policy bundle, CSV, JSON, and marker files

**Testing**: Boost unit tests, native DI smoke binaries, tracer evidence script, MiniNDN availability checks, negative checks

**Target Platform**: Ubuntu/Linux development host and MiniNDN topology

**Project Type**: Mixed C++ runtime/examples plus Python experiment scripts

**Performance Goals**: Correct timing capture, not throughput optimization

**Constraints**: Preserve dynamic API, NAC-ABE/security path, one-time tokens, MiniNDN-first validation, and no generated service/stub compatibility

**Scale/Scope**: One toy native DI service with `/Backbone`, two `/Head/Shard/*` providers, and `/Merge`; at least two provider assignment layouts

## Constitution Check

- **Canonical Dynamic Runtime**: Pass. This feature continues the generic native DI path and does not reintroduce generated stubs.
- **Security Is Part Of The Data Path**: Pass. Full MiniNDN topology work must preserve controller permissions, NAC-ABE, and token checks.
- **CodeGraph First, Source Verified**: Pass. Existing MiniNDN and native provider symbols were explored through CodeGraph first.
- **Spec-Driven Changes**: Pass. This feature has Spec Kit artifacts before implementation.
- **Verify With The Right Scope**: Pass. Local unit/smoke tests gate code changes; MiniNDN is the intended network validation surface.

## Project Structure

### Documentation (this feature)

```text
specs/002-native-di-minindn-tracer/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   ├── evidence.md
│   ├── assignments.md
│   └── llm-gate.md
├── checklists/
│   └── requirements.md
└── tasks.md
```

### Source Code (repository root)

```text
examples/python/NDNSF-DistributedInference/native_di_tracer/
├── native_tracer_policy.yaml
├── plan_tracer.py
├── run_minindn_tracer.sh
└── artifacts/

examples/
├── DI_NativePlanManifestSmoke.cpp
├── DI_NativePlanSchemaSmoke.cpp
└── DI_NativeProviderSessionSmoke.cpp
```

**Structure Decision**: Keep the user-facing tracer command in the native tracer example directory and keep evidence docs in Spec Kit. A full topology launcher can be added later under `Experiments/` after this evidence contract is stable.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | N/A |
