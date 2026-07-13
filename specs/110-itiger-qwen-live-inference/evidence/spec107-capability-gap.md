# Spec 107 Capability Gap at Spec 110 Start

Snapshot date: 2026-07-13. Spec 107 has 34 unchecked tasks. Its completed
foundation supplies identity, bounded state, timing, evidence, and fault
contracts, but it has not completed the measured generation-session branch.

## Blocking generation path

| Tasks | Missing capability | Source owner |
|---|---|---|
| T025-T027 | Once-only attribution runs and a digest-bound bottleneck decision | `tools/ndnsf-di/run_spec107_attribution.py`, `Experiments/NDNSF_DI_LlmPipeline_Minindn.py` |
| T028-T031 | RED session tests and bounded token-epoch/KV/final-once implementation | `tests/unit-tests/di-qwen-generation-session.t.cpp`, `NDNSF-DistributedInference/cpp/ndnsf-di/QwenGenerationSession.cpp` |
| T032 | Collaboration dependency I/O plus lease/attempt authority | `NdnsfCollaborationDependencyIo.cpp`, `NativeProviderRuntime.cpp` |
| T033 | One user generation request instead of acceptance-mode per-token requests | `examples/python/NDNSF-DistributedInference/llm_pipeline/user.py` |
| T034 | Real provider wiring and `qwen-generation-session-v1` readiness | `examples/DI_NativeProviderExecutable.cpp` |
| T035-T042 | Candidate wiring, exact tokens, and three 60-second MiniNDN repetitions | experiment script and Spec 107 evidence |

## Later open gates

- T048-T053: live fault cells and recovery verdict.
- T060-T065: clean canaries, operations, and conditional soak.
- T068-T072: final regressions, successor manifest, Spec 106 handoff, and
  strict closeout.

Spec 110 may implement the missing reusable session/runtime behavior in these
source owners, but it must not relabel a Spec 110 iTiger result as a completed
Spec 107 MiniNDN campaign. Spec 107 once-only results and identities remain
independent and immutable.
