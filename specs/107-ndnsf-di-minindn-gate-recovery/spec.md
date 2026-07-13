# Feature Specification: NDNSF-DI MiniNDN Gate Recovery

**Feature Branch**: `Experimental`

**Created**: 2026-07-12

**Status**: Draft — Independent Successor Candidate to Frozen Spec 105

**Input**: Freeze Spec 105 and create an independent MiniNDN performance,
live-fault-injection, and operations revision with a new candidate identity.

## Scope and Frozen Boundary

This feature produces a new local MiniNDN candidate without modifying,
reclassifying, pooling, or replacing any Spec 105 task, evidence file, result
directory, threshold, or release decision. Spec 105 remains a completed
negative result with `minindnCandidateOverall=BLOCK` and
`physicalProductionOverall=DEFERRED`.

Spec 107 owns only the next local candidate. Its candidate namespace is
`spec107-c1`; the final candidate ID is mechanically derived from that namespace
plus the source, profile, model, plan, artifact, and lineage-lock digests. All
diagnostic, performance, recovery, canary, operations, and soak records use
separate campaign IDs and unique output directories. No Spec 105 campaign ID or
output path may be reused.

The supported workload remains deliberately unchanged:

- Qwen2.5-0.5B-Instruct at the frozen model revision;
- three contiguous CPU ONNX stages on the same three MiniNDN provider nodes;
- the same prompt, tokenizer, greedy decoding, batch size, maximum 512 input
  tokens, and exactly 32 generated tokens for acceptance;
- one offered generation per second for a 60-second measured window;
- normal NDNSF permission, NAC-ABE, token, replay, provider-permission, attempt,
  lease, digest, and deadline checks;
- local MiniNDN only. Physical hosts, CUDA evidence, real production identities,
  and physical release authority remain in Spec 106.

Diagnosis may use shorter or single-request cells, but diagnostic output is
permanently ineligible for acceptance. Exactly one measured bottleneck branch
may be implemented after attribution. The feature must reuse existing V2,
collaboration, Targeted, lease, large-data, and execution-attempt primitives;
it must not add a new Core wire name merely to improve this application.

## User Scenarios & Testing

### User Story 1 - Attribute and Remove the Dominant Delay (Priority: P1)

An evaluator can identify where one generated token spends its time and can see
that the new candidate removes the largest measured avoidable delay without
changing the workload or acceptance threshold.

**Why this priority**: Spec 105 completed only 25 of 60 generations and measured
a p95 20.17 times the matched baseline. Optimizing without causal attribution
would invite parameter tuning and another uninterpretable campaign.

**Independent Test**: Run non-acceptance warm and cold diagnostic cells, account
for request admission, ACK/selection, plan/lease validation, stage queueing,
stage compute, tensor encode/decode, dependency fetch/publish, response, and
inter-token time, and verify that one declared branch explains and removes the
largest avoidable component.

**Acceptance Scenarios**:

1. **Given** the frozen Spec 105 lineage and an unmodified workload, **When** a
   diagnostic cell runs, **Then** at least 99% of completed token steps have a
   complete, non-overlapping critical-path decomposition whose sum reconciles
   with end-to-end time within 5% or 10 ms, whichever is larger.
2. **Given** competing optimization hypotheses, **When** attribution closes,
   **Then** exactly one branch is selected by a preregistered dominance rule and
   all rejected branches and measurements remain recorded.
3. **Given** an optimization changes request behavior, **When** its contract
   tests run, **Then** permissions, tokens, replay protection, leases, attempt
   authority, digests, deadlines, and bounded queues still fail closed.

---

### User Story 2 - Pass a New Fixed-Load Qwen Candidate (Priority: P1)

An application user receives correct 32-token Qwen generations from the new
three-stage MiniNDN candidate at the unchanged fixed load and latency threshold.

**Why this priority**: A local deployment candidate cannot advance while the
fixed workload is slower than its declared capacity class.

**Independent Test**: After correctness, resource, disk, and campaign-identity
preflight, execute exactly three new 60-second repetitions at 1 offered
generation per second and compare each repetition independently with the frozen
matched single-node baseline.

**Acceptance Scenarios**:

1. **Given** the new candidate artifacts and the frozen prompt, **When** any
   generation completes, **Then** all 32 tokens exactly match the frozen greedy
   baseline and provider evidence matches the candidate identity.
2. **Given** one 60-second repetition, **When** it closes, **Then** at least 99%
   of offered generations complete, achieved throughput is at least 0.95
   generation/s, and distributed p95 is at most 2.0 times the matched baseline
   p95; all three repetitions must pass independently.
3. **Given** insufficient disk, memory, identity, telemetry, or artifact state,
   **When** preflight runs, **Then** the campaign does not start and emits a
   retained `INVALID_PREFLIGHT` record rather than a partial repetition.

---

### User Story 3 - Prove Recovery with Live MiniNDN Faults (Priority: P1)

An operator can inject a real fault into owned MiniNDN child processes or their
data path and observe one bounded recovery or one exact terminal failure without
stale authority.

**Why this priority**: Spec 105 verified recovery contracts but recorded
`networkInjection=false`; contract fixtures cannot prove live distributed
behavior.

**Independent Test**: Execute the preregistered live fault matrix once per cell
against isolated MiniNDN processes, retaining process identity, provider boot,
injection, control, attempt, terminal, cleanup, and security evidence.

**Acceptance Scenarios**:

1. **Given** a selected provider process is killed and restarted, **When** one
   replacement is allowed, **Then** the provider boot changes, the old attempt
   cannot become authoritative, and exactly one replacement or terminal result
   is observed before the original deadline.
2. **Given** a straggler, missing segment, digest mismatch, stale telemetry,
   cache eviction, restarted provider, or late old output, **When** its live cell
   executes, **Then** the declared injection is proven on the MiniNDN path and
   produces the exact bounded recovery or terminal reason.
3. **Given** any live fault cell ends, **When** cleanup runs, **Then** all owned
   child processes, waits, leases, temporary routes, and attempt state are gone;
   host/default NFD and unrelated processes were never targeted.

---

### User Story 4 - Operate and Soak the Accepted Local Candidate (Priority: P2)

An operator can install, start, inspect, restart, upgrade, roll back, stop, and
soak the accepted candidate through documented commands and structured evidence.

**Why this priority**: Performance and recovery are necessary but not sufficient
for a reproducible local deployment candidate.

**Independent Test**: After performance and live recovery pass, execute two
clean-directory canaries, live process supervision and restart, N-to-N+1
upgrade, N+1-to-N rollback with Repo preservation, and one unreplaced 24-hour
1 RPS soak with one scheduled provider restart.

**Acceptance Scenarios**:

1. **Given** two clean local staging directories, **When** the runbook is
   followed, **Then** both deployments reach ready and complete the canary
   without source edits, hidden commands, or reused mutable runtime state.
2. **Given** a running candidate, **When** it is inspected, restarted, upgraded,
   rolled back, and stopped, **Then** structured status and metrics retain exact
   release, plan, evidence, provider boot, queue, request, terminal, and Repo
   identities.
3. **Given** passing preflight gates, **When** the 24-hour soak executes once,
   **Then** at least 99% of requests complete, every completed result is correct,
   no security bypass or unbounded resource growth occurs, and the scheduled
   restart creates no duplicate terminal authority.

---

### User Story 5 - Trust the Successor Release Decision (Priority: P2)

An evaluator can mechanically distinguish a complete Spec 107 candidate from
Spec 105 and can decide whether Spec 106 may begin.

**Why this priority**: A new campaign must not erase the prior negative result or
silently promote incomplete evidence.

**Independent Test**: Tamper with lineage, campaign identity, evidence digests,
dimension status, or physical status and verify the release gate fails closed.

**Acceptance Scenarios**:

1. **Given** any Spec 105 file or digest differs from the lineage lock, **When**
   the Spec 107 gate runs, **Then** it blocks without rewriting either feature.
2. **Given** any correctness, performance, recovery, application-security, or
   local-operations artifact is missing or failed, **When** the gate runs,
   **Then** `minindnCandidateOverall=BLOCK`.
3. **Given** every local dimension passes, **When** the gate runs, **Then** it
   emits a digest-bound Spec 107 candidate manifest with
   `minindnCandidateOverall=PASS` while keeping
   `physicalProductionOverall=DEFERRED`; only then may Spec 106 consume it.

### Edge Cases

- A diagnostic result appears faster only because warmup entered the measured
  window, logging changed, fewer tokens were generated, or a shorter timeout
  discarded slow requests.
- ACK role coverage is already early-closing, so the initial timeout hypothesis
  is falsified and another component dominates.
- Targeted tokens or cached assignments are exhausted, stale, provider-misbound,
  replayed, or invalidated by provider restart.
- One provider reports correct compute evidence but a different candidate,
  plan, artifact, boot, or telemetry epoch.
- A fault controller attempts to kill a PID that was not spawned in the current
  campaign process group.
- A fault fires before the positive control proves the request reached the
  intended stage, or fires after the request already terminated.
- The filesystem has enough space for artifacts but not for the projected soak
  logs; preflight must account for bounded sampling and safety margin.
- A result directory already exists, is root-owned, or is being written by a
  stale process.
- A late output from attempt N arrives after N+1 is authoritative.
- Metrics export fails while request correctness remains intact; the evidence
  dimension blocks because observability is incomplete.
- The local service supervisor is not PID 1; evidence must say
  `local-process-supervision`, not physical systemd acceptance.
- A performance or fault cell fails after starting; it is retained and is not
  silently retried or replaced.

## Requirements

### Functional Requirements

- **FR-001**: Spec 107 MUST treat commit
  `48877b5854aa9231d7b28f423160e5695388fce4`, the Spec 105 task digest,
  release-gate digest, performance-evidence digest, and recovery-evidence digest
  as an immutable lineage lock, and MUST fail if any locked value changes.
- **FR-002**: Spec 107 MUST NOT modify any file under
  `specs/105-ndnsf-di-deployment-readiness/` or any retained Spec 105 result.
- **FR-003**: Every Spec 107 artifact MUST carry the `spec107-c1` namespace and a
  final candidate ID derived from source, profile, model, plan, artifact, and
  lineage-lock digests; Spec 105 IDs and output paths MUST be rejected, and a
  mixed provider/user set lacking the exact generation-session capability MUST
  fail preflight rather than fall back inside an eligible campaign.
- **FR-004**: Diagnostic, correctness, performance, fault, canary, operations,
  soak, and release-gate campaigns MUST have separate identities, output roots,
  commands, stop rules, and eligibility labels.
- **FR-005**: Diagnostic runs MUST be permanently ineligible for acceptance and
  MUST retain all successful, negative, invalid, and rejected hypotheses.
- **FR-006**: Timing evidence MUST decompose admission, ACK/selection,
  plan/lease validation, queueing, compute, tensor encode/decode, dependency
  fetch/publish, response, and inter-token time with the shared stable sampler.
- **FR-007**: The bottleneck decision MUST select exactly one optimization branch
  whose avoidable component is largest and accounts for at least 25% of warm
  end-to-end token-step time; if no branch meets this rule, implementation MUST
  stop for replanning rather than combine speculative optimizations.
- **FR-008**: The selected optimization MUST reuse existing unified service
  names, V2 messages, collaboration, Targeted, lease, attempt, large-data, and
  security primitives; a new Core wire name is prohibited.
- **FR-009**: The fixed Qwen model revision, tokenizer, prompt, greedy decoding,
  batch size, three contiguous stage count, 512-input-token limit, 32-output-token
  acceptance length, and single-node baseline MUST remain unchanged.
- **FR-010**: Every completed distributed generation MUST match the frozen
  baseline token-by-token; mismatch MUST terminate the cell and block the candidate.
- **FR-011**: Generation, request, wait, callback, token-pair, assignment, tensor,
  and metrics queues MUST remain bounded and expose occupancy and rejection reasons.
- **FR-012**: Warmup MUST finish outside the measured window; TRACE MUST remain
  disabled for acceptance, while sampled timeline diagnostics may be enabled only
  in explicitly labeled diagnostic cells.
- **FR-013**: Performance acceptance MUST execute exactly three unique 60-second
  repetitions at 1 offered generation/s, with zero automatic retries or
  replacement repetitions, and each repetition MUST pass independently.
- **FR-014**: Performance evidence MUST retain offered/completed/failed/unfinished
  counts, throughput, p50/p95/p99, TTFT, inter-token latency, decomposition,
  queue progress, host/process resources, exact tokens, and provider evidence.
- **FR-015**: Binary artifacts MUST be materialized once in a content-addressed,
  read-only local store; measured repetitions MUST reference rather than copy or
  re-export them, and reproducible `.pt` intermediates MUST not be retained.
- **FR-016**: Before any measured or soak campaign, preflight MUST verify unique
  unwritten output paths, no stale writers, artifact hashes, ownership, and free
  space greater than the projected new output plus a 1 GiB safety reserve.
- **FR-017**: The live MiniNDN fault controller MUST act only on child processes,
  routes, delays, or data owned by the current campaign and MUST record
  `networkInjection=true`, target identity, injection time, observed effect, and cleanup.
- **FR-018**: The live fault matrix MUST cover provider kill/restart, straggler,
  missing segment, dependency digest mismatch, stale telemetry, KV eviction,
  provider boot change, and late old output, with one execution per cell.
- **FR-019**: Recovery MUST permit at most one replacement attempt, preserve the
  original deadline, issue authenticated cancel/supersede control, reject old
  authority, and emit exactly one final success or stable terminal reason.
- **FR-020**: Fault cleanup MUST prove bounded threads, waits, leases, routes,
  processes, and state after every cell; cleanup failure blocks subsequent cells.
- **FR-021**: Local operations MUST execute real packaged commands and child
  processes in isolated staging, while labeling the evidence
  `local-process-supervision`; physical PID-1/systemd acceptance remains Spec 106.
- **FR-022**: Canary, restart, upgrade, rollback, and soak MUST preserve
  authoritative Repo/catalog state and discard only identity-incompatible
  disposable model, activation, and KV caches.
- **FR-023**: The 24-hour soak MUST execute once at 1 RPS only after performance,
  correctness, security, recovery, disk, and canary preflight pass, and MUST
  include one preregistered provider restart.
- **FR-024**: INFO logs, metrics, evidence, and bundles MUST not expose prompts,
  payloads, tensors, KV values, token values, secrets, or private keys; expected
  tokens may appear only as count and digest.
- **FR-025**: The release gate MUST verify all evidence paths and SHA-256 digests,
  derive local dimension verdicts mechanically, retain Spec 105 as BLOCK, and
  keep `physicalProductionOverall=DEFERRED`.
- **FR-026**: Spec 106 MUST remain deferred unless the digest-bound Spec 107
  candidate reports `minindnCandidateOverall=PASS`; Spec 107 cannot claim
  physical security, GPU, cross-host, or production-supervisor evidence.

### Key Entities

- **Lineage Lock**: Exact Spec 105 commit and artifact digests that prove the
  predecessor was frozen rather than rewritten.
- **Candidate Identity**: Digest-bound Spec 107 source, workload, profile, model,
  plan, artifacts, and lineage used by every eligible result.
- **Timing Decomposition**: Reconciled critical-path intervals and sampled event
  evidence for one generation and token step.
- **Bottleneck Decision**: Immutable record of hypotheses, dominance calculation,
  selected single branch, rejected branches, and falsification condition.
- **Campaign Preflight**: Eligibility decision covering output exclusivity,
  process ownership, hashes, resources, disk projection, and gate prerequisites.
- **Live Fault Cell**: One injection target, trigger, observed effect, recovery or
  terminal result, authority proof, and cleanup proof.
- **Local Operations Record**: Install, start, inspect, restart, upgrade,
  rollback, stop, Repo preservation, and supervision classification.
- **Successor Release Gate**: Evidence manifest and per-dimension PASS/BLOCK with
  a separate physical DEFERRED status.

## Success Criteria

### Measurable Outcomes

- **SC-001**: The lineage lock verifies all five frozen Spec 105 identifiers and
  zero files under the Spec 105 feature or retained results change.
- **SC-002**: At least 99% of completed diagnostic token steps have a timing sum
  within 5% or 10 ms of observed end-to-end time, and exactly one branch meets
  the 25% dominance rule before implementation begins.
- **SC-003**: All correctness cells and every completed acceptance generation
  match all 32 expected tokens exactly, with zero optimistic or mixed provider evidence.
- **SC-004**: Each of three independent 60-second repetitions completes at least
  99% of 60 offered generations, achieves at least 0.95 generation/s, and has
  p95 no greater than 2.0 times the frozen matched single-node p95.
- **SC-005**: All eight live fault cells record `networkInjection=true`, prove the
  intended effect, and end with exactly one authoritative result or declared
  terminal reason and zero second replacement attempts.
- **SC-006**: After each fault cell, owned process, thread, wait, lease, route, and
  attempt counts return to their declared baseline bounds; zero unrelated host
  process or host/default NFD is modified.
- **SC-007**: Two clean-directory canaries independently reach ready and pass
  correctness, identity, status, metrics, stop, and cleanup checks without source edits.
- **SC-008**: Upgrade and rollback complete in isolated local supervision with
  zero authoritative Repo/catalog loss, zero trusted incompatible cache, and
  exact release/plan/evidence bindings before and after rollback.
- **SC-009**: The single 24-hour 1 RPS soak has at least 99% completion, zero
  incorrect outputs, zero security bypasses, zero unbounded resource growth,
  and exactly one preregistered restart with no duplicate terminal authority.
- **SC-010**: Every measured/soak campaign either passes preflight before any
  role starts or retains one `INVALID_PREFLIGHT` record; no campaign reaches
  `ENOSPC`, reuses an output directory, or re-exports model artifacts per repetition.
- **SC-011**: The final gate reports `minindnCandidateOverall=PASS` only when all
  local dimensions and evidence digests pass; otherwise it reports BLOCK, while
  `physicalProductionOverall` remains DEFERRED in every case.

## Assumptions

- The local host, MiniNDN topology, three CPU ONNX artifacts, and frozen matched
  baseline remain available.
- The current code already contains early collaboration role-coverage selection,
  Targeted token batches, bounded generation scheduling, bounded dependency waits,
  attempt epochs, and local packaging primitives; Spec 107 first measures their
  actual contribution instead of duplicating them.
- The current 4.9 GiB free-space state is sufficient only if artifacts are reused
  and logs/timelines are sampled; preflight makes the actual decision.
- A failed new campaign is a valid completed result and keeps the successor gate
  BLOCK; the feature does not promise that optimization must make the local CPU
  meet the unchanged capacity target.

## Explicit Non-Goals

- Editing, reopening, passing, or replacing Spec 105.
- Physical machines, CUDA, GPU telemetry, cross-host routing, real production
  identities, second-operator validation, or physical systemd/PID-1 acceptance.
- Changing the model, prompt, tokenizer, output length, offered load, measured
  duration, thresholds, deadline, retry count, or matched baseline to obtain PASS.
- Adding Kubernetes, autoscaling, multi-tenancy, a fourth provider node, or a new
  generic Core wire protocol.
- Pooling repetitions, deleting failures, silently retrying, or treating
  diagnostic results as acceptance evidence.
