# Implementation Plan: NDNSF-DI MiniNDN Gate Recovery

**Branch**: `Experimental` | **Date**: 2026-07-12 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from
`specs/107-ndnsf-di-minindn-gate-recovery/spec.md`

## Summary

Freeze Spec 105 byte-for-byte and create a new `spec107-c1` candidate that
keeps its workload, thresholds, security path, three-node MiniNDN topology, and
physical deferral. The shortest code route is to replace 32 independent
per-token collaboration requests with one generation-scoped collaboration
session: participant selection, security setup, assignment, scope material, and
attempt authority are established once, while the three existing Qwen ONNX
roles iterate token epochs through existing collaboration data topics and
provider-local KV state. A mandatory diagnostic gate must first confirm that
repeated request orchestration remains the dominant avoidable component; if the
attribution is falsified, implementation stops and the Spec Kit artifacts are
replanned rather than accumulating multiple optimizations.

The same candidate then executes an independently preregistered live fault
matrix using only harness-owned MiniNDN child processes and a separate fault
provider executable, followed by local process-supervised canary,
upgrade/rollback, and soak. A mechanical release gate preserves Spec 105 BLOCK,
requires all Spec 107 evidence digests, and keeps physical production DEFERRED.

## Technical Context

**Language/Version**: C++17; Python 3.8; POSIX shell

**Primary Dependencies**: NDNSF generic dynamic runtime, ndn-cxx/NFD, ndn-svs,
MiniNDN, pybind11 wrapper, ONNX Runtime 1.26 CPU, NumPy, existing Qwen tokenizer
and artifact exporter, systemd-compatible packaging

**Storage**: Content-addressed read-only ONNX artifact directory under ignored
`results/spec107-artifacts/`; unique ignored campaign result directories;
tracked Spec Kit summaries and evidence manifests; authoritative Repo state
unchanged

**Testing**: Boost.Test C++ unit suite; Python `unittest`; shell syntax and
packaging staging checks; MiniNDN correctness/performance/live-fault/canary/soak
campaigns; ASan/UBSan or documented supported analysis

**Target Platform**: Current local Ubuntu x86_64 development host running
MiniNDN and CPU ONNX only

**Project Type**: C++ runtime library plus Python orchestration, experiment
harness, CLI/packaging, and Spec Kit evidence

**Performance Goals**: In each of three independent 60-second cells, at least
99% of 60 offered generations complete, throughput is at least 0.95 generation/s,
and p95 is at most 2.0 times the frozen 6,854.20 ms matched baseline p95

**Constraints**: Exact 32-token correctness; one generation is the request unit;
one replacement attempt; original deadline retained; no new Core wire name; no
threshold/workload/timeout/retry changes; sampled timelines; artifact reuse; no
physical or host/default-NFD acceptance claims

**Scale/Scope**: One user, one controller, three MiniNDN provider nodes, three
contiguous Qwen stages, four concurrent generation workers, 60 offered
generations per repetition, eight live fault cells, two clean canaries, one
24-hour soak

## Constitution Check

### Pre-design gate

| Principle | Result | Reason |
|---|---|---|
| I. Canonical Dynamic Runtime | PASS | Unified service names, V2 collaboration, Targeted terminology, and existing generic APIs remain; no generated/static API returns. |
| II. Security Is Part Of The Data Path | PASS | Permission, NAC-ABE, one-time tokens, replay, provider permission, lease, attempt, digest, and deadline checks remain mandatory in diagnostics and acceptance. |
| III. CodeGraph First | PASS | CodeGraph and exact source/result inspection established current scheduling, early role coverage, Targeted support, and missing live fault injection. |
| IV. Spec-Driven Durable Work | PASS | Spec 107 owns the new algorithm/evaluation/operations work and preserves an immutable predecessor. |
| V. Verify With The Right Scope | PASS | Final network/performance/security checks use MiniNDN; physical evidence stays in Spec 106; measured windows remain 60 seconds. |

### Post-design gate

PASS. The design adds no new Core protocol, debug authorization bypass, physical
claim, or alternative source of authority. Qwen generation-session behavior is
owned in the DI application layer. Experimental fault behavior is isolated in a
separate executable and cannot be enabled in the production provider binary.

## Evidence-Led Architecture

### Frozen predecessor

`lineage-lock.json` pins:

- frozen repository commit `48877b5854aa9231d7b28f423160e5695388fce4`;
- Spec 105 tasks SHA-256
  `4dff3d74337b35fba0677b933ecf9b8ac6d745f64bb0d6ab453bb5d1916a26bf`;
- release gate SHA-256
  `2752ca1853b5243099dd40dd07ef86d80f24d34dbe6e6c91d567e13ecef296f9`;
- performance evidence SHA-256
  `1090503b7fe58c127aa83187ea0a15f50053fe77dab5aa746780aaf797d39364`;
- recovery evidence SHA-256
  `2777b17ddc231b910667fb4866222359c51f8d9ceb8043a2fa873f8bee66d257`.

Spec 107 tooling reads these files but has no write path to Spec 105. A final
`git diff --exit-code 48877b5 -- specs/105-...` is insufficient by itself
because later unrelated history may contain the files; the exact locked file
digests are authoritative.

### Preliminary performance attribution

The retained valid Spec 105 run shows:

- one 32-token correctness request: 25,320.02 ms;
- fixed-load completed p50/p95: 93,094.66/138,227.90 ms;
- three-stage service EWMA near 1 + 35.33 + 62.32 ms per token;
- repeated `NDNSF_DI_CLIENT_INFERENCE_TIMING request_ms` values around
  0.4–1.5 seconds for individual token steps;
- one new collaboration request, scope-key operation, assignment, and final
  response for every token.

This makes repeated per-token orchestration the selected preliminary branch:
it is several times larger than observed compute and is repeated 32 times. The
first implementation phase adds a reconciled timeline and a locked
`bottleneck-decision.json`. Implementation proceeds only if request/session
orchestration remains the largest avoidable component and at least 25% of warm
token-step time. Existing early role-coverage selection means the change is not
“reduce ACK timeout”; it removes repeated session establishment.

### Generation-scoped collaboration session

One application request carries the frozen prompt/context, maximum token count,
candidate/plan/attempt bindings, and Qwen generation-session specification.
Normal collaboration ACK/selection chooses all three role providers once. The
selected providers then retain the same collaboration session and iterate:

```text
token epoch N
  Stage 0 consumes prompt/token delta + local KV -> hidden N
  Stage 1 consumes hidden N + local KV         -> hidden N
  Stage 2 consumes hidden N + local KV         -> logits/token N
  Stage 2 publishes token N to Stage 0 feedback topic
  repeat until 32 tokens or terminal condition
```

The loop uses existing collaboration data publication/fetch, key scopes,
attempt epoch, execution lease, dependency digest, and final response. The
feedback topic is application data under the existing collaboration namespace,
not a new Core wire name. Token epoch appears in every dependency identity and
KV binding. The final response contains only the generated token bundle and
evidence; intermediate responses are not authoritative.

The new DI components are:

- `QwenGenerationSessionSpec`: immutable request limits and identity bindings;
- `QwenGenerationSession`: per-provider bounded token loop and state machine;
- generation-session codec inside DI application payloads;
- user-side one-request orchestration and progress accounting;
- provider-side final/feedback publication and exact terminal reasons.

The existing per-token user loop remains available only as a labeled diagnostic
baseline. It cannot satisfy the Spec 107 release gate.

### Ownership and authority

| Concern | Owner | Authority boundary |
|---|---|---|
| Permission, NAC-ABE, tokens, replay, collaboration selection, V2 naming | Existing NDNSF Core | Unchanged and authoritative; DI cannot bypass it |
| Token epochs, provider-local KV, feedback, generation terminal | DI C++ `QwenGenerationSession` | Application state inside one authorized collaboration attempt |
| Offered-load scheduling, exact-token oracle, progress accounting | Qwen Python user | Advisory/client evidence; cannot create provider authority |
| Process topology and live injection | MiniNDN harness plus separate fault binary | May affect only current campaign-owned children/data |
| Candidate/preflight/evidence/release digests | `tools/ndnsf-di/` | Mechanical experiment eligibility; cannot claim physical readiness |
| Install/start/restart/rollback commands | Packaging plus local supervisor | Local process evidence only; Spec 106 owns PID-1/physical proof |

There is one terminal authority: the current execution attempt inside the
authorized generation session. Telemetry, cached KV, fault triggers, and client
progress are never authority sources.

### Candidate migration and rollback

The generation-session mode is candidate-scoped and capability-gated. Providers
advertise a digest-bound `qwen-generation-session-v1` readiness capability;
mixed providers or users that lack the exact capability fail preflight instead
of falling back inside an eligible campaign. The existing per-token path remains
unchanged as a diagnostic control during Spec 107 and is excluded by the release
gate. No persisted Repo/catalog schema is migrated; generation/KV state is
disposable and candidate/boot/attempt bound.

Before measured evidence, rollback is a normal source revert plus a new
candidate identity. After a campaign is preregistered, its source and manifest
are immutable: failure is retained, and any code change requires a new successor
candidate rather than replacing a Spec 107 cell. Operations rollback restores
the prior package and Repo/catalog identity, then discards incompatible
generation/KV/cache state before readiness.

### Recovery and replacement

Live failure evidence uses two provider executable classes:

- the normal `DI_NativeProviderExecutable` for performance and kill/restart;
- a separate `DI_NativeFaultProviderExecutable` linked to the same DI runtime
  but with an experiment-only injection adapter for straggler, drop, corruption,
  stale telemetry, cache eviction, and late-output cells.

The production executable contains no fault flag. The harness creates a
campaign process registry with PID, process-group ID, `/proc` start time,
provider identity, boot identity, role, and command digest. A destructive action
is rejected unless all fields match the currently owned child. Provider loss or
cache loss permits one replacement; the replacement uses a new attempt epoch
and either compatible state or a full-context rebuild. Old attempts, provider
boots, segments, KV state, and final responses remain non-authoritative.

The eight fault cells are separate, execute once, and do not depend on the
performance gate. Cleanup failure stops later cells. Fault evidence never counts
as performance acceptance.

### Operations and soak

`packaging/ndnsf-di-systemd/run-local-supervised.sh` and a small Python process
registry execute the exact packaged production commands without pretending the
local supervisor is systemd PID 1. Static unit hardening remains validated.
Local operations cover clean install, doctor, start, readiness, structured
status/metrics, restart, N/N+1 activation, rollback, Repo preservation, stop,
and cleanup. Performance and recovery can proceed independently; the 24-hour
soak requires every earlier local gate to pass.

### Artifact and disk discipline

The three ONNX files are imported once into
`results/spec107-artifacts/<artifact-set-digest>/` using same-filesystem
hardlinks or reflinks when safe and a verified copy otherwise. The store becomes
read-only after hash verification. `.pt` export intermediates are deleted before
campaign eligibility. Repetitions reference the store and never export or copy
models.

Preflight computes projected artifact delta, logs, CSV/JSON evidence, sampled
timeline, and soak growth. Required free bytes are:

```text
projected new bytes + 1 GiB safety reserve
```

An existing/nonexclusive output directory, stale writer, ownership mismatch,
hash mismatch, or insufficient free space produces one `INVALID_PREFLIGHT`
record before MiniNDN roles start.

## Experiment Design (ARS plan mode)

### Material Passport

- Origin Skill: experiment-agent
- Origin Mode: plan
- Origin Date: 2026-07-12
- Verification Status: UNVERIFIED
- Version Label: spec107_code_plan_v1

### Research question and hypothesis

**RQ**: Can generation-scoped collaboration remove repeated orchestration cost
enough for the unchanged local three-stage CPU MiniNDN candidate to pass fixed
performance, live recovery, and operations gates without weakening security or
evidence integrity?

**Primary hypothesis**: Replacing 32 per-token collaboration sessions with one
generation session removes the dominant avoidable control-plane cost while
preserving exact token output and bounded authority.

The hypothesis is falsified if attribution does not meet the 25% dominance
rule, any token differs, any security/authority invariant fails, or any of the
three independent performance cells misses a threshold.

### Variables

| Class | Variables |
|---|---|
| Independent | Per-token baseline vs generation-scoped candidate; live fault type; normal vs restart operation |
| Primary dependent | Per-repetition completion, achieved throughput, p95 ratio, exact-token correctness |
| Secondary dependent | TTFT, inter-token latency, timing decomposition, queue depth, stage rate, RSS, available memory, recovery/cleanup time |
| Controlled | Source/candidate digests, model/tokenizer/prompt, three stage artifacts, topology, roles, 1 RPS, 60 seconds, 32 tokens, INFO logging, warmup, timeout, retry/replacement bounds |
| Confounds recorded | CPU frequency/load, available memory, filesystem space, NFD/MiniNDN versions, process boot IDs, background processes, artifact cache state |

### Cells and ordering

1. Lineage/artifact/disk preflight.
2. Two non-acceptance attribution cells: one warm 32-token generation and one
   four-generation concurrency cell; no automatic repeat.
3. Unit/contract/security correctness after generation-session implementation.
4. Exactly three independent performance cells; all must pass individually.
5. Eight once-only live fault cells plus one positive control.
6. Two clean canaries and local operations drills.
7. One 24-hour soak with one scheduled restart, only after every prerequisite passes.

### Analysis

- Acceptance is threshold-based and per repetition; results are never pooled to
  rescue a failing repetition.
- Report p50/p95/p99 and nonparametric 95% bootstrap intervals descriptively;
  intervals do not replace hard thresholds.
- Compare tokens deterministically by exact sequence and digest.
- Reconcile critical-path timing for each sampled token step.
- Report all failures, invalid preflights, unfinished requests, and cleanup failures.
- Perform the project's 11 distributed-systems fallacy scan plus survivor,
  confirmation, optional-stopping, and instrumentation-effect checks.

### Stop rules

- Stop before implementation if the selected branch fails dominance or timing reconciliation.
- Stop a campaign on token mismatch, evidence/identity mismatch, security bypass,
  unowned process target, cleanup failure, insufficient disk, or malformed output.
- Retain a started failing cell; never rerun or replace it automatically.
- A failed performance cell blocks soak but does not erase separately executed
  live recovery evidence.

## Project Structure

### Documentation (this feature)

```text
specs/107-ndnsf-di-minindn-gate-recovery/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── lineage-lock.json
├── experiment-plan.md
├── quickstart.md
├── contracts/
│   ├── candidate-lineage.md
│   ├── timing-and-bottleneck.md
│   ├── generation-session.md
│   ├── live-fault-record.md
│   └── successor-release-gate.md
├── checklists/requirements.md
├── evidence/
└── tasks.md
```

### Source Code

```text
NDNSF-DistributedInference/
├── cpp/ndnsf-di/
│   ├── QwenGenerationSession.hpp
│   └── QwenGenerationSession.cpp
└── ndnsf_distributed_inference/
    ├── qwen_pilot.py
    ├── client.py
    ├── app.py
    └── operations.py

examples/
├── DI_NativeProviderExecutable.cpp
├── DI_NativeFaultProviderExecutable.cpp
└── python/NDNSF-DistributedInference/llm_pipeline/
    ├── user.py
    └── provider.py

Experiments/
└── NDNSF_DI_LlmPipeline_Minindn.py

packaging/ndnsf-di-systemd/
└── run-local-supervised.sh

tools/ndnsf-di/
├── spec107_candidate.py
├── spec107_lineage.py
├── spec107_identity.py
├── spec107_preflight.py
├── spec107_artifacts.py
├── spec107_timing.py
├── run_spec107_attribution.py
├── run_spec107_performance.py
├── spec107_fault_controller.py
├── run_spec107_live_faults.py
├── spec107_local_supervisor.py
├── run_spec107_operations.py
└── build_spec107_release_bundle.py

tests/
├── unit-tests/distributed-inference-async-runtime.t.cpp
├── unit-tests/di-qwen-generation-session.t.cpp
└── python/
    ├── test_ndnsf_di_spec107_*.py
    └── test_ndnsf_di_deployment_readiness.py
```

**Structure Decision**: Keep generic transport/security unchanged. Put the
Qwen-specific iterative state machine in the existing DI C++ layer, orchestration
and candidate validation in Python, live-only injection in a separate example
binary, and campaign control in the existing MiniNDN harness.

## Delivery Phases

1. Freeze lineage, candidate schema, artifact store, preflight, and timing contracts.
2. Add failing generation-session and identity/security tests.
3. Execute attribution; confirm or block the selected architecture.
4. Implement and validate the one-request Qwen generation session.
5. Add and execute live MiniNDN fault injection independently of performance.
6. Execute the fixed performance campaign once.
7. Execute local canary/operations and, only after all gates pass, the soak.
8. Generate the successor gate, audit, converge, synchronize docs, and update
   Spec 106 prerequisite only if the candidate passes.

## Complexity Tracking

No constitution violation requires justification. The new
`QwenGenerationSession` is application-owned and replaces repeated application
orchestration; it does not duplicate generic Core selection or create a second
security/protocol authority.
