# Specification Quality Checklist: NDNSF-DI iTiger Distributed Qwen Execution

**Purpose**: Validate the corrected requirements before planning and live work
**Created**: 2026-07-13
**Feature**: [spec.md](../spec.md)

## Intent fidelity

- [x] The primary outcome is real NDNSF-DI distributed inference, not standalone Qwen
- [x] Complete iTiger environment provisioning is in scope
- [x] The first candidate is one node, three Provider processes, three GPUs, and one NFD
- [x] A separately keyed 0.5B extension requires at least two nodes and one cross-node NDN dependency
- [x] All requested Qwen2.5-Instruct sizes are present
- [x] Standalone inference is only an oracle/baseline

## Requirement quality

- [x] Requirements are testable and avoid implementation placeholders
- [x] Pre-start blockers cannot close live execution tasks
- [x] Post-start failures are retained as negative experimental outcomes
- [x] GPU proof is stronger than device visibility
- [x] Security, identity, storage, teardown, and authority boundaries are explicit
- [x] 60-second repetitions, warmup, metrics, and sample thresholds are explicit
- [x] Framework overhead and placement/network delta use two distinct matched contrasts
- [x] A durable pre-submit journal closes the `sbatch` acknowledgement-loss window
- [x] No unresolved clarification marker remains

## Architecture and ownership

- [x] Slurm owns allocation lifecycle and Apptainer owns container execution
- [x] NFD/NDN owns inter-node transport; NDNSF owns security and invocation
- [x] NDNSF-DI owns stages, dependencies, generation sessions, and evidence
- [x] A frozen Slurm process map binds ranks, GPUs, identities, NFD sockets, readiness, and teardown
- [x] One unprivileged NFD is owned by each unique allocated node
- [x] Specs 107/108 are reused without treating unchecked work as delivered
- [x] Spec 106 retains physical-production authority

## Notes

- This checklist validates requirements only. It does not claim that the image,
  multi-node network, generation session, Qwen exports, or GPU experiments exist.
