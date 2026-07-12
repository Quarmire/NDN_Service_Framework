# Implementation Plan: NDNSF-DI Physical Production Pilot

**Branch**: `Experimental` | **Date**: 2026-07-12 | **Spec**: [spec.md](spec.md)

## Summary

Deploy one immutable passing Spec 105 candidate on exactly three physical Ubuntu
GPU nodes and collect the evidence unavailable in MiniNDN. This feature changes
deployment profiles and acceptance evidence only. Any runtime or algorithm
change returns to Spec 105 and creates a new candidate release.

## Technical Context

**Runtime**: Existing Spec 105 C++/Python binaries and operator CLI  
**Platform**: Three Ubuntu x86_64 NVIDIA GPU nodes with NFD and systemd  
**Security**: Real identities, trust schema, NAC-ABE, permissions, one-time tokens  
**Workload**: Frozen Qwen2.5-0.5B three-stage, <=512 input, 32 output, batch one  
**Measurement**: Matched canary plus 24-hour 1 RPS soak, INFO only  
**Storage**: Authoritative Repo state preserved; model/KV/activation caches disposable

## Entry Gate

Implementation cannot begin until:

1. Spec 105 has no unchecked tasks;
2. `minindnCandidateOverall=PASS`;
3. candidate source/release/profile/plan/model/artifact digests are frozen;
4. three compatible physical nodes and a second operator are available;
5. the production campaign is registered before observing results.

## Constitution Check

- **Canonical dynamic runtime**: PASS. Spec 106 consumes the existing V2 dynamic
  runtime and introduces no generated, split-name or Direct API.
- **Security in the data path**: PASS. Real permission, NAC-ABE, token, replay and
  provider-permission checks are mandatory and have negative cells.
- **CodeGraph/source verification**: PASS. Any future profile or doctor edit must
  start from current callers and finish with targeted tests.
- **Spec-driven durable work**: PASS. Physical acceptance has its own requirements,
  tasks, evidence, stop conditions and rollback rather than weakening Spec 105.
- **Right validation scope**: PASS. Spec 106 is explicitly physical and cannot
  substitute MiniNDN evidence for real-host security or operations.
- **GSD/ARS**: PASS. The deferred 24-hour campaign has immutable controls,
  reproducibility records and resumable state.

## Architecture and Ownership

- Spec 105 owns code, algorithms, packaging and local candidate evidence.
- Spec 106 owns physical profiles, host preflight, real identities, cross-host
  routes, physical campaigns and `physicalProductionOverall`.
- Core security and DI attempt/lease invariants remain unchanged.
- A physical finding requiring source changes is returned to Spec 105; Spec 106
  never patches the candidate in place.

## Execution Sequence

1. Verify immutable candidate and hardware inventory.
2. Freeze three-host profile, routes, identities, devices and safety limits.
3. Install twice from clean hosts and close doctor/readiness.
4. Run positive and negative production-security cells.
5. Run matched physical single-node/distributed canaries.
6. Run provider restart and same-three-node fallback cells.
7. Run N->N+1 upgrade and N+1->N rollback twice.
8. Run one 24-hour 1 RPS soak with prespecified restart.
9. Generate the production release gate without replacement runs.

## Validation and Stop Rules

- Stop on candidate digest drift, wrong output, security bypass, corrupt Repo
  state, stale attempt authority, unbounded growth or hardware safety limit.
- Retain the failed run; never silently rerun it.
- Missing node/operator/identity/soak evidence is BLOCK, not estimated PASS.
- Matched comparisons keep model, prompts, backend, logging, security, request
  schedule, duration and hardware allocation fixed.

## Rollback

Stop submissions, drain or cancel within the candidate bound, activate the
previous digest-verified release, delete incompatible disposable caches, restart
in dependency order, run doctor/security/correctness canaries, and preserve all
failed-release evidence. Authoritative Repo objects are never rewritten.

## Project Structure

```text
specs/106-ndnsf-di-physical-pilot/
├── spec.md
├── plan.md
├── experiment-plan.md
├── quickstart.md
├── traceability.md
├── tasks.md
├── contracts/release-handoff.md
├── checklists/requirements.md
└── evidence/
```

Physical profiles and evidence will use:

```text
packaging/ndnsf-di-systemd/config/physical/
results/spec106-physical-<cell>-<unique>/
specs/106-ndnsf-di-physical-pilot/evidence/
```
