# Implementation Plan: DI Runtime-Aware User-Side Planner

**Branch**: `047-di-runtime-aware-user-planner` | **Date**: 2026-07-05 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/047-di-runtime-aware-user-planner/spec.md`

## Summary

Keep NDNSF-DI planning in the user process, but make each runtime assignment depend on provider ACKs that report real resource state, DI fragment residency, provider-to-provider network metrics, and short-lived lease offers. Split the design into reusable NDNSF core mechanisms and NDNSF-DI semantics: core provides generic ACK metadata, generic admission leases, selection lease validation, provider runtime hints, and peer telemetry; DI defines model fragments, GPU/CPU/disk residency, KV-cache locality, and graph-placement cost. The planner treats distributed inference as graph placement: each role/provider mapping has node cost, and each dependency between roles has provider-pair edge cost.

## Technical Context

**Language/Version**: C++17 for NDNSF runtime and native provider/user paths; Python 3 for planner helpers, campaign scripts, and fixtures.

**Primary Dependencies**: Existing NDNSF dynamic runtime, NDN-SVS, ndn-cxx, MiniNDN, current NDNSF-DI NativeTracer/LLM planner artifacts.

**Storage**: In-memory provider runtime state and lease tables for MVP; JSON/CSV result files for campaign evidence; no persistent database required.

**Testing**: Python unit/regression tests, C++ smoke tests where message/schema parsing changes are introduced, focused shell regressions, and MiniNDN campaigns.

**Target Platform**: Ubuntu/Linux NDNSF development environment and MiniNDN.

**Project Type**: Distributed systems runtime/library plus experiment harnesses.

**Performance Goals**: Under multi-user contention, avoid selecting invalid/stale resources, reduce avoidable poor provider-pair transfers, and expose p50/p95 latency and utilization evidence.

**Constraints**: Preserve user-side planner MVP, existing V2 naming, NAC-ABE routing, UserToken/ProviderToken replay protection, provider permissions, and legacy ACK compatibility where runtime-aware mode is not required. Keep model-layer, GPU, KV-cache, and fragment-residency semantics out of NDNSF core.

**Scale/Scope**: First target is small LLM/NativeTracer MiniNDN workloads with 3-5 providers and multiple simultaneous users; design must generalize to larger provider sets through bounded ACK payloads and optional metric fetches.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **Canonical Dynamic Runtime**: PASS. Feature extends generic dynamic service invocation and DI planning; it does not reintroduce generated stubs or split service/function APIs.
- **Security Is Part Of The Data Path**: PASS. Lease/admission must preserve NAC-ABE, permission, UserToken/ProviderToken, replay checks, and provider permissions.
- **CodeGraph First, Source Verified**: PASS. Initial exploration used CodeGraph; implementation tasks require source verification before edits.
- **Spec-Driven Changes For Durable Work**: PASS. This feature uses Spec Kit artifacts before implementation.
- **Verify With The Right Scope**: PASS. Tasks include unit/schema tests plus MiniNDN validation for network/performance behavior.

## Project Structure

### Documentation (this feature)

```text
specs/047-di-runtime-aware-user-planner/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   ├── ack-runtime-state.md
│   ├── lease-selection.md
│   └── planner-metrics.md
├── checklists/
│   └── requirements.md
└── tasks.md
```

### Source Code (repository root)

```text
ndn-service-framework/
├── ServiceUser.*                  # generic lease-aware selection hooks
├── ServiceProvider.*              # generic lease/admission validation hooks
├── messages.* / message helpers   # generic ACK metadata, lease, peer telemetry envelope support
└── ...                            # existing security/token paths must remain intact

NDNSF-DistributedInference/
└── ndnsf_distributed_inference/
    ├── planner/runtime-aware planner modules
    ├── provider/DI runtime state and fragment inventory helpers
    └── runtime_v1.py

examples/python/NDNSF-DistributedInference/native_di_tracer/
├── provider profile and fragment fixture files
├── rate/campaign/search helpers
└── planner evidence generation

Experiments/
└── NDNSF_DI_*MiniNDN*.py           # multi-user and asymmetric network campaign hooks

tests/python/
└── test_*runtime_aware_planner*.py
```

**Structure Decision**: Put reusable admission, lease, metadata, reason-code, and peer-telemetry envelopes in the NDNSF C++ framework. Put model fragment keys, fragment residency, KV-cache locality, DI graph placement, and DI evaluation logic in NDNSF-DI Python/native paths. Do not create a standalone planner service in this feature.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | N/A |
