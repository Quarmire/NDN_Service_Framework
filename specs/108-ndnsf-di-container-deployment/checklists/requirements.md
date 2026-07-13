# Specification Quality Checklist: NDNSF-DI OCI Deployment Adapters

**Purpose**: Validate specification completeness and quality before implementation planning
**Created**: 2026-07-12
**Feature**: [spec.md](../spec.md)

## Content quality

- [x] Operator outcomes and evidence authority are explicit
- [x] OCI build source is separated from runtime adapters
- [x] Docker Compose and Slurm + Apptainer responsibilities do not overlap
- [x] All mandatory specification sections are complete
- [x] No implementation claim is presented as already complete

## Requirement completeness

- [x] No unresolved clarification markers remain
- [x] Functional requirements are individually testable
- [x] Success criteria are measurable
- [x] All six user stories have independent tests and acceptance scenarios
- [x] Edge cases include scheduler, GRES, version, storage, network, evidence, and fallback failures
- [x] Scope, assumptions, dependencies, and exclusions are explicit
- [x] iTiger storage lifetime and quota semantics are explicit
- [x] GPU GRES, physical device, and container mapping are distinguished
- [x] OCI digest and SIF checksum are both required
- [x] Multi-node iTiger networking is fail-closed until measured

## Architecture and authority

- [x] Adapters remain thin and do not duplicate planner/runtime/security logic
- [x] Existing systemd deployment remains the rollback surface
- [x] Docker GPU and Apptainer GPU prerequisites are correctly separated
- [x] Substrate evidence is separated from candidate evidence
- [x] Physical-production authority remains in Spec 106
- [x] Current job 145855 is labeled preliminary substrate evidence only

## Task readiness

- [x] Every FR and SC maps to tasks and an acceptance surface
- [x] Tasks use exact target paths and dependency order
- [x] Tests precede corresponding implementation within each phase
- [x] Live jobs are bounded, unique, and prohibit automatic rerun of failures
- [x] Documentation, audit, rollback, and Spec 106 handoff tasks are included

## Notes

- Runtime names such as Docker, Slurm, Apptainer, GRES, CUDA, and ONNX Runtime are necessary platform constraints, not premature application design.
- The requirements permit an iTiger substrate PASS but never infer candidate inference or physical-production readiness from GPU visibility alone.
