# Spec 107 Pre-implementation Audit

## Verdict

`CONDITIONAL PASS`

The revision is ready for lineage, test, instrumentation, and attribution work
(T001–T027). It is not yet authorized to implement the proposed generation
session or run acceptance campaigns: the current evidence supports the
orchestration hypothesis but does not yet provide a reconciled critical path.
T027 is the controlling gate. A falsified or non-unique `<25%` result requires
replanning, not implementation. No Critical or High finding remains in the
documents.

## Findings

| ID | Severity | Dimension | Location | Finding | Required action |
|---|---|---|---|---|---|
| AUD-001 | MEDIUM | Evidence / performance | `plan.md:105`; `llm_pipeline/user.py:189`; Spec 105 `telemetry-performance-check.md:34` | Code fact: the user currently issues a collaboration request inside the token loop. Evidence fact: Spec 105 p50/p95 are 93,094.66/138,227.90 ms. The generation-session design is still a proposed causal remedy, not measured proof. | Execute T021–T027. Proceed to T028 only if one reconciled avoidable component is uniquely largest and >=25%; otherwise set `REPLAN_REQUIRED`. |
| AUD-002 | MEDIUM | Operations / validity | `spec.md:261`; `plan.md:222`; `tasks.md:T005–T008,T063` | Environment fact: only 4.9 GiB is currently free. Repeated artifact copies or an ungated soak can invalidate evidence through ENOSPC. | Implement projected-growth preflight and content-addressed artifact reuse before any measured role starts. A failed preflight is a retained terminal record, not permission to delete evidence or reduce the workload. |
| AUD-003 | LOW | Recovery / security | `research.md:60`; `tasks.md:T043–T047`; Spec 105 `fault-recovery.md:26` | Evidence fact: predecessor recovery is contract-only and explicitly has `networkInjection=false`. The new fault executable does not yet exist, so live recovery remains proposed. | Keep injection in the separate fault binary, prove the normal provider has no fault flag, and require owned PID/PGID/start-time/boot/identity matching plus cleanup before each next cell. |
| AUD-004 | LOW | Migration | `plan.md:174`; `tasks.md:T071` | Spec 106 still consumes the frozen predecessor until a real successor PASS exists. Updating it now would imply authority that Spec 107 has not earned. | Preserve current deferral; update only the prerequisite wording after T069 PASS. On BLOCK, retain Spec 106 unchanged and record the skipped task. |

## Traceability Gaps

| Source/Requirement/Task | Missing link | Impact |
|---|---|---|
| None | All 26 FRs and 11 SCs map to evidence-producing tasks in `traceability.md`. | None |

No mechanism is unowned: Core retains transport/security authority; the DI C++
layer owns generation epochs/KV/terminal state; the Python user owns scheduling
and correctness evidence; the MiniNDN harness and separate fault provider own
injection; packaging owns local lifecycle; Spec 106 owns physical proof.

## Readiness Scorecard

| Dimension | Ready? | Notes |
|---|---|---|
| Intent and scope | YES | Spec 105 is immutable; Spec 107 is independent and MiniNDN-only. |
| Architecture and ownership | CONDITIONAL | Ownership is explicit; chosen optimization remains gated by T027. |
| Security/correctness | YES FOR IMPLEMENTATION | Existing permission/token/replay/lease/attempt invariants remain and have RED negative-test tasks. |
| Task executability | YES | 72 sequential tasks name files, dependencies, evidence, and stop rules. |
| Validation/evidence | CONDITIONAL | Design is rigorous; no Spec 107 measurements exist yet. |
| Migration/rollback | YES | Capability mismatch fails preflight; no Repo schema migration; campaigns are immutable. |
| Code reality | YES FOR T001–T027 | Current token loop, Targeted support, frozen negative performance, and contract-only recovery were verified through CodeGraph and exact reads. |

## Metrics

- User stories: 5
- Functional requirements: 26
- Success criteria: 11
- Tasks: 72 (0 complete)
- Requirement coverage: 26/26 FRs and 11/11 SCs
- Unmapped tasks: 0
- Placeholders: 0
- Findings: 0 Critical, 0 High, 2 Medium, 2 Low

## Assumptions And Evidence Limits

- No Spec 107 source implementation, build, unit test, MiniNDN campaign, live
  injection, canary, or soak was executed during this planning audit.
- The 4.9 GiB disk snapshot is current only for this audit and must not replace
  per-campaign projected-growth preflight.
- CodeGraph was up to date (7,166 files; 159,014 nodes), but broad queries were
  noisy because vendored gRPC sources are indexed; exact repository reads were
  used to confirm the controlling paths.
- Preliminary request timings are retained Spec 105 evidence. They justify
  attribution work, not a claim that generation-scoped collaboration will pass.
- Physical systemd/PID-1, GPU, cross-host, production identity, and operator
  evidence remain outside scope and DEFERRED to Spec 106.

## Next Actions

1. Execute T001–T020 to create fail-closed lineage, identity, disk, artifact,
   timing, state, security, and evidence contracts.
2. Execute T021–T027 exactly once and freeze `bottleneck-decision.json`.
3. If and only if T027 selects the generation-session branch at >=25%, start
   T028. Otherwise formally revise Spec 107 before changing runtime behavior.
4. Do not start model materialization, measured performance, fault campaigns,
   or soak while their individual preflight is incomplete or disk projection
   fails.
