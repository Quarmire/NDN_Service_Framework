# Spec 105 to Spec 106 Physical-Work Migration Record

**Date**: 2026-07-12  
**Status**: COMPLETE AND DEFERRED

## Ownership Decision

Spec 105 owns the local MiniNDN candidate: CPU ONNX correctness and performance,
generation scheduling, Linux host/process telemetry, plan validity, bounded
recovery, local packaging, local staging, and MiniNDN operations evidence.

Spec 106 owns only evidence that needs physical hosts or a second operator:

- three physical NVIDIA GPU nodes and CUDA/device-UUID evidence;
- real identities and production-strength trust/security validation;
- cross-host NFD routes and physical network behavior;
- physical GPU telemetry and clock-skew checks;
- clean-host second-operator reproduction;
- physical restart, upgrade/rollback, canary, and 24-hour soak evidence;
- authority to change `physicalProductionOverall` from `DEFERRED`.

## Entry Gate

No Spec 106 implementation or measurement may begin until all of the following
are true:

1. Spec 105 has no unchecked tasks;
2. `minindnCandidateOverall=PASS` for an immutable release;
3. source, release, profile, plan, model, and artifact digests are frozen;
4. three compatible physical nodes and a second operator are available;
5. the physical campaign is preregistered.

The failed initial Spec 105 1 RPS campaign does not move algorithm, scheduler, or
capacity repair into Spec 106. Those remain Spec 105 work under Revision R1.

## Migration Audit

- Physical requirements/tasks present in Spec 105: none; FR-024 explicitly
  defers them.
- Local CPU/Linux tasks present in Spec 106: none except consuming and verifying
  the immutable candidate contract.
- Duplicate release authority: none. Spec 105 owns `minindnCandidateOverall`;
  Spec 106 alone owns `physicalProductionOverall`.
- Current physical status: `DEFERRED`, not BLOCK and not PASS, because hardware
  and operator prerequisites are unavailable and the candidate entry gate is not
  yet satisfied.

