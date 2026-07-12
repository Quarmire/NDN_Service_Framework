# Spec 105 Pre-Implementation Audit

**Date**: 2026-07-12  
**Scope**: `spec.md`, `plan.md`, `tasks.md`, supporting contracts and the current
NDNSF-DI code/results  
**Evidence state**: design audit only; no Spec 105 implementation or acceptance
campaign has executed

## Verdict

`CONDITIONAL PASS`

The feature is fit for phased implementation, beginning with T001-T008 and the
evidence-truth MVP. It follows the shortest credible deployment route: one small
model, one CUDA backend, one three-stage topology, MiniNDN before hardware, and
systemd before a larger control plane. No CRITICAL or HIGH design defect was
found, all 24 functional requirements and 10 success criteria have task
coverage, and the strict structural audit passes. Implementation is conditional
on resolving two frozen-profile wording conflicts at T006 and treating Qwen KV
export as a hard feasibility checkpoint before later runtime integration. No
production-readiness claim is permitted until T098 mechanically passes all six
release dimensions.

## Findings

| ID | Severity | Dimension | Location | Finding | Required action |
|---|---|---|---|---|---|
| A-001 | MEDIUM | Security / cross-artifact consistency | `quickstart.md:39-53`; `experiment-plan.md:65-76`; `plan.md:247-258` | The MiniNDN quickstart calls the acceptance workload “real security” while the same section and controlling experiment plan correctly state that the dummy-keychain run is not cryptographic-strength evidence. An implementer could incorrectly close the security dimension from MiniNDN. | In T006, encode MiniNDN security as `application-auth-path-executed` and keep the cryptographic security dimension BLOCKED until the physical real-identity cell. T047/T048 must not emit a full security PASS. |
| A-002 | MEDIUM | Intent / deployment scope | `spec.md:17-24`; `plan.md:43-45`; `tasks.md:17,150` | The product boundary requires fallback roles on the same three GPU nodes, but the plan scale line says “optional standby” nodes. Extra standby hardware would broaden the claimed shortest route and make fault evidence incomparable. | T006 must freeze exactly three GPU nodes with preprovisioned fallback roles on those nodes. Additional standby nodes are out of Spec 105 acceptance and require a later experiment profile. |
| A-003 | MEDIUM | Technical feasibility / validation order | `research.md:35-61`; `tasks.md:84-95` | Multi-token Qwen serviceability depends on exporting and validating per-stage KV inputs/outputs. Existing evidence proves a one-token ONNX forward, not the proposed bounded decode/KV contract. T037/T038 correctly expose this unknown, but it must be a hard checkpoint rather than a late implementation surprise. | Complete T037 and T038 before T039-T044. If stable three-stage KV artifacts cannot be produced and matched, stop and re-plan the execution boundary; do not fall back to synthetic compute or silently weaken 32-token acceptance. |
| A-004 | LOW | Traceability automation | `traceability.md:3-57`; `tasks.md:12-199` | Requirement coverage is complete, but 32 setup, schema, implementation, documentation, or closure task IDs are not written literally in the requirement table because the table uses selected IDs/ranges and semantic grouping. They are justified by phases/stories, but a literal task-to-requirement checker reports them as unlisted. | At T008, emit a machine-readable task-to-requirement map or add an explicit cross-cutting row. Do not delete the tasks; none was found unnecessary. |

## Traceability Gaps

| Source/Requirement/Task | Missing link | Impact |
|---|---|---|
| T003-T004, T007-T008 and other cross-cutting tasks | Not every task ID is explicitly enumerated in `traceability.md` | Low audit-automation precision; no functional coverage gap |
| T037-T038 to later US2 work | Sequential IDs imply the dependency, but there is no named phase checkpoint between export proof and runtime integration | Wasted work if KV export is infeasible; controlled by A-003 |

All FR-001 through FR-024 and SC-001 through SC-010 have closing tasks. No
unjustified runtime mechanism was found: execution evidence prevents false
claims; typed tensors/KV enable bounded generation; measured telemetry prevents
configured capacity from masquerading as live state; the bounded scheduler
prevents thread growth; attempt epochs prevent stale authority; packaging and
release gates deliver the operator outcome.

## Readiness Scorecard

| Dimension | Ready? | Notes |
|---|---|---|
| Intent and scope | Conditional | Narrow route is correct; T006 must exclude extra standby nodes |
| Architecture and ownership | Yes | DI owns model, telemetry policy, wait scheduling and attempts; Core wire names and security remain unchanged |
| Security/correctness | Conditional | Invariants and negative tests are strong; MiniNDN/physical evidence labels must remain distinct |
| Task executability | Yes | 102 sequential, path-specific tasks; test-first gates and stop conditions are explicit |
| Validation/evidence | Conditional | Experimental controls are rigorous; Qwen KV export and all physical evidence remain unverified |
| Migration/rollback | Yes | Additive evidence migration, zero-reader removal, disposable caches and authoritative Repo preservation are explicit |
| Code reality | Yes for implementation | Proposed work targets verified current gaps rather than claiming they are already implemented |

## Code-Reality Findings

- `Experiments/NDNSF_DI_NativeTracer_Minindn.py:3130-3135` forces the
  deterministic runner for `llm-proportional`, while lines 3262-3268 assign
  `runnerMode=qwen-onnx-native`. This validates US1 as the controlling first gate.
- `examples/DI_NativeProviderExecutable.cpp:235-262` implements that runner as a
  configured sleep plus synthetic 1x1 float tensors; lines 787-806 register it
  under the ONNX backend name. Existing affected throughput is control/dataflow
  evidence, not Qwen compute evidence.
- `ProviderRoleWorker.cpp:183-216` appends one `std::thread` per pending input set
  to `m_inputWaiters`; the destructor joins all of them at lines 67-83. The
  bounded scheduler is necessary and correctly owned by DI.
- `runtime_v1.py:2064-2095` calculates prefill/decode duration algebraically, and
  production-named run/sweep paths call it at lines 2655-2669 and 2709 onward.
  The CLI migration tasks do not duplicate a current real provider adapter.
- `packaging/ndnsf-di-systemd/`, `release_gate.py`, and `qwen_pilot.py` do not yet
  exist. They are accurately labeled proposed and have creation tasks; the plan
  does not misstate them as implemented.
- Existing Core `GenericAdmissionLease` remains execution authority. Spec 105's
  `PlanLease` is an advisory feasibility/validity record and explicitly cannot
  override admission, avoiding a second authorization source.

## Metrics

- User stories: 5
- Functional requirements: 24
- Success criteria: 10
- Tasks: 102 (23 parallel-marked; 0 complete)
- Requirement coverage: 34/34, 100%
- Semantically unmapped tasks: 0
- Literally unlisted task IDs in the traceability table: 32
- Placeholders: 0
- Critical / High / Medium / Low findings: 0 / 0 / 3 / 1
- Strict Spec Kit structure: PASS
- Constitution conflicts: 0

## Assumptions and Evidence Limits

- The CodeGraph index was current at audit time: 2,151 files, 47,589 nodes and
  159,779 edges.
- No Spec 105 implementation test, MiniNDN acceptance run, physical-node canary,
  restart drill, upgrade/rollback drill, or 24-hour soak has run.
- Three compatible Ubuntu NVIDIA GPU nodes, a second operator, supported CUDA /
  ONNX Runtime versions and real identity/trust setup are external prerequisites.
- Existing real-Qwen evidence is an anchor, not proof of 32-token cached decode,
  measured placement, recovery, systemd operation or production security.
- The physical release remains BLOCKED if hardware or any mandatory evidence is
  unavailable; missing evidence is never converted into a waiver or estimate.

## Next Actions

1. Execute T001-T006, preserving the historical evidence mismatch and freezing
   the exact same-three-node profiles with correct security labels.
2. Close A-001/A-002 and record the preflight/audit state in T007-T008 before any
   runtime edit.
3. Implement T009-T030 as MVP 1. Do not optimize against historical deterministic
   throughput.
4. Prove T037/T038 before continuing past the Qwen export boundary. Failure here
   requires re-planning, not acceptance weakening.
5. Continue only through each hard checkpoint; physical nodes begin after T078.
6. Treat T098 as the only production-pilot PASS authority. Until then the honest
   deployment status remains `NOT READY` or `BLOCK`, depending on the report.
