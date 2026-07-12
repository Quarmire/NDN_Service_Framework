# Spec 105 Revision R1 Audit

**Date**: 2026-07-12  
**Scope**: Failed T047-T048 campaign, current load-driver code, revised Spec 105
requirements/plan/tasks, and the Spec 105/106 ownership boundary  
**Evidence status**: Code-inspected and result-inspected; no replacement campaign run

## Verdict

`BLOCK` for the current MiniNDN candidate release. `CONDITIONAL PASS` to resume
implementation at T049 under Revision R1.

The original 1 RPS candidate cannot pass because all three repetitions completed
0/60 generations and latency/resource acceptance is incomplete. Continuing the
unchanged campaign is prohibited. Implementation may resume because Revision R1
retains that failure, adds a deterministic measurement-validity repair before any
new campaign, moves the resource implementation ahead of revised acceptance, and
keeps physical production work exclusively in Spec 106.

## Findings

| ID | Severity | Dimension | Location | Finding | Required action |
|---|---|---|---|---|---|
| R1-001 | HIGH | Validation / code reality | `examples/python/NDNSF-DistributedInference/llm_pipeline/user.py:166-235`; `NDNSF-DistributedInference/ndnsf_distributed_inference/client.py:141-150,627-665` | Each token callback submits the next token to one four-worker FIFO executor while 60 sessions are offered. Later sessions' token work can remain ahead of an earlier session's next token, so zero completed generations does not isolate CPU capacity from queue discipline. | T049-T051 must prove generation-level ownership, bounded admission, progress/queue accounting, and preregister a new campaign identity before measurement. |
| R1-002 | HIGH | Evidence integrity | `evidence/qwen-minindn-performance.md:28-72` | All three initial runs are decisive negative evidence for the tested system: 0/180 complete, undefined latency percentiles, and missing resource metrics. Partial token subrequests cannot be promoted to completed application requests. | Retain the runs unchanged and keep H3/SC-002/SC-003 BLOCK for the original candidate. Never pool them with a revised campaign. |
| R1-003 | HIGH | Task executability | Original T048 evidence; revised `tasks.md:114-127` | The initial campaign requested resource evidence before the resource probe tasks existed. The original result correctly reports the metric absent, but that ordering could never close the gate. | T052-T061 now implement and validate resource/plan evidence before revised acceptance T062. |
| R1-004 | MEDIUM | Performance architecture | `user.py:166-212`; `spec.md:38-54` | One 32-token application generation performs 32 sequential distributed collaboration calls. This is a real property of the tested design and may still fail at 1 RPS after scheduler repair. | Do not predict a pass. T062 must report the complete system result; failure keeps the candidate BLOCKED without threshold changes. |
| R1-005 | MEDIUM | Scope / ownership | `spec.md:13-34,284-294`; `specs/106-ndnsf-di-physical-pilot/plan.md:21-53` | Local CPU ONNX, MiniNDN algorithms, scheduler, telemetry and packaging belong to Spec 105. Physical CUDA/device, real identities, cross-host routes and second-operator evidence belong to Spec 106. | Keep Spec 106 deferred until Spec 105 has no unchecked work, a passing candidate manifest, and hardware/operator availability. |

## Traceability Gaps

| Source/Requirement/Task | Missing link | Impact |
|---|---|---|
| Original T047-T048 campaign | No generation-scheduler validity requirement existed | Zero-complete result was valid for the system but causally underdetermined |
| Original T048 resource metric | Resource probes were scheduled after the gate | Mandatory evidence was structurally unavailable |
| Physical acceptance language in older Spec 105 audit | Predated the local-only revision | Historical audit is not current authority; Spec 106 now owns that work |

Revision R1 closes these document gaps through FR-025/FR-026, SC-011, T049-T051,
the revised T062 campaign, and the explicit T094 preflight stop rule.

## Readiness Scorecard

| Dimension | Ready? | Notes |
|---|---|---|
| Intent and scope | Yes | Shortest route stays local MiniNDN; physical work remains deferred |
| Architecture and ownership | Conditional | Ownership is clear; generation protocol may still miss capacity |
| Security/correctness | Yes to continue | T046 correctness passed; no security threshold is weakened |
| Task executability | Yes | Revised campaign now follows scheduler and resource evidence work |
| Validation/evidence | Conditional | Original failure is sound; revised driver/campaign are not executed |
| Migration/rollback | Yes | No persisted or wire schema migration is introduced by this revision |
| Code reality | Yes | Finding is grounded in the actual callback and executor path |

## Experiment Validation

- Overall confidence: `RED_FLAG` for a capacity PASS; `SOLID` for the negative
  claim that the tested candidate did not service 1 RPS.
- Statistical content: zero successes among 180 offered requests; application
  latency distributions are undefined and must remain unreported.
- Fallacy coverage: 11/11 retained in `qwen-minindn-performance.md`. The revision
  specifically prevents survivorship bias, post-result threshold movement,
  replacement-run selection, and single-cause attribution.
- Reproducibility: no rerun performed by design. The three original unique result
  directories and exact frozen settings remain the canonical failed evidence.

## Metrics

- User stories: 5
- Functional requirements: 26
- Success criteria: 11
- Tasks: 102 (48 complete, 54 pending at revision)
- Structural audit before revision: PASS
- Critical / High / Medium / Low findings: 0 / 3 / 2 / 0
- Physical tasks remaining in Spec 105: 0

## Assumptions and Evidence Limits

- The CodeGraph index was current at audit time: 2,156 files, 47,742 nodes and
  160,874 edges.
- FIFO ordering follows Python's current `ThreadPoolExecutor` work-queue behavior;
  deterministic T049 fixtures must establish the project-level invariant rather
  than relying on an undocumented scheduling hope.
- No corrected scheduler, telemetry probe, revised campaign, recovery matrix,
  packaging drill, or soak has executed yet.
- A scheduler repair can improve measurement validity without making the CPU
  profile meet 1 RPS. No capacity improvement is claimed.

## Next Actions

1. Execute T049 test-first and preserve the failing breadth-first fixture.
2. Implement T050 without changing timeout, retry, offered load, token count, or
   the underlying security/collaboration path.
3. Close T051 with deterministic evidence and a preregistered new campaign ID.
4. Continue T052-T061 so T062 has the resource and plan evidence absent at T048.
5. Run T062 exactly once as specified. A failure keeps release BLOCK and causes
   T094 to record `NOT RUN / BLOCK`; it does not stop completion of independent
   implementation, audit, and documentation tasks.

