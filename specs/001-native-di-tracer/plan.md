# Implementation Plan: Native DI Tracer

**Branch**: `001-native-di-tracer` | **Date**: 2026-06-24 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/001-native-di-tracer/spec.md`

## Summary

Build a native NDNSF-DI tracer-bullet before expanding to full LLM workloads. The feature records a minimal distributed inference policy, proves that Python-generated `native-execution-plan.json` is consumed by C++, executes assigned native provider roles, captures readiness/artifact/timing evidence, and stages MiniNDN validation as the final acceptance surface.

## Technical Context

**Language/Version**: C++17 runtime and examples; Python 3 for policy generation and orchestration scripts

**Primary Dependencies**: NDNSF dynamic runtime, ndn-cxx/NFD, MiniNDN for network validation, existing NDNSF-DI C++ native execution layer

**Storage**: Filesystem result directories for generated policy bundles, logs, cache entries, timing CSV, and summaries

**Testing**: Existing C++ Boost unit tests, C++ smoke examples under `examples/DI_*`, Python policy generation checks, MiniNDN experiment script when available

**Target Platform**: Ubuntu/Linux development host and MiniNDN topology

**Project Type**: Mixed C++ library/examples plus Python orchestration package

**Performance Goals**: Tracer prioritizes correctness and evidence; timing fields must be captured even when toy workloads are used

**Constraints**: Preserve generic dynamic runtime API, V2 naming, NAC-ABE permission routing, one-time token checks, and MiniNDN-first validation

**Scale/Scope**: One small service with 2-3 providers and enough roles to exercise source input, dependency edge, intermediate publish, fan-in, and final response

## Constitution Check

- **Canonical Dynamic Runtime**: Pass. The tracer uses current dynamic NDNSF-DI paths and does not reintroduce generated service/stub APIs.
- **Security Is Part Of The Data Path**: Pass. Runtime security is not bypassed; collaboration service flow, permissions, tokens, and NAC-ABE routing remain intact.
- **CodeGraph First, Source Verified**: Pass. Current DI symbols were explored through CodeGraph before edits.
- **Spec-Driven Changes**: Pass. This feature is captured in Spec Kit artifacts before implementation.
- **Verify With The Right Scope**: Pass. Unit/smoke tests gate code changes; MiniNDN remains final network validation.

## Project Structure

### Documentation (this feature)

```text
specs/001-native-di-tracer/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── llm-planner-followup.md
├── contracts/
│   ├── tracer-policy.md
│   ├── native-plan.md
│   └── evidence.md
├── checklists/
│   └── requirements.md
└── tasks.md
```

### Source Code (repository root)

```text
NDNSF-DistributedInference/
├── ndnsf_distributed_inference/
│   ├── policy.py
│   └── planner_registry.py
└── cpp/ndnsf-di/
    ├── NativeExecutionPlanJson.*
    ├── NativeArtifactMaterializer.*
    ├── NativeProviderReadiness.*
    ├── NativeProviderHandler.*
    ├── NativeProviderSession.*
    └── ProviderRoleWorker.*

examples/
├── DI_NativePlanSchemaSmoke.cpp
├── DI_NativePlanManifestSmoke.cpp
├── DI_NativeProviderExecutable.cpp
└── python/NDNSF-DistributedInference/native_di_tracer/

tests/unit-tests/
└── distributed-inference-async-runtime.t.cpp
```

**Structure Decision**: Keep tracer implementation inside existing NDNSF-DI package, C++ examples, and unit test files. Use `specs/001-native-di-tracer/` for durable feature design and acceptance state.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | N/A |
