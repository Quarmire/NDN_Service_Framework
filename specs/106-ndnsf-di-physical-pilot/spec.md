# Feature Specification: NDNSF-DI Physical Production Pilot

**Feature Branch**: `Experimental`

**Created**: 2026-07-12

**Status**: Deferred Until Hardware Is Available

**Input**: Consume a passing Spec 105 MiniNDN deployment candidate and perform
the physical-node, real-identity, cross-host and production-operations evidence
that cannot be obtained on the local MiniNDN host.

## Scope and Boundary

Spec 106 is an acceptance and operations feature, not an algorithm redesign. It
deploys one immutable Spec 105 candidate release on exactly three Ubuntu x86_64
NVIDIA GPU nodes, using one primary Qwen stage per node and preprovisioned
fallback roles on the same nodes. It owns real identities, trust configuration,
cross-host NDN routing, physical telemetry, second-operator reproduction,
restart/upgrade/rollback drills and a 24-hour soak.

Spec 106 MUST NOT repair failed Spec 105 correctness, performance, scheduler or
recovery gates by changing algorithms, timeouts, retries, workload or artifacts.
A failed prerequisite returns work to Spec 105 with a new candidate release.

## User Scenarios & Testing

### User Story 1 - Prove Production Security on Real Identities (Priority: P1)

An operator deploys the immutable candidate with real controller, provider,
user and Repo identities and proves certificate validation, NAC-ABE permissions,
tokens, replay protection and provider permissions without a dummy keychain.

**Independent Test**: Execute positive bootstrap/inference plus forged,
misbound, replayed and unauthorized negative cells on the three physical nodes.

**Acceptance Scenarios**:

1. Valid identities and policies complete permission bootstrap and one bounded
   Qwen request through all three providers.
2. Invalid trust, token replay, provider-token replay, wrong artifact digest or
   wrong attempt epoch fails closed with a stable reason.
3. No key, prompt, token, tensor or KV payload bytes appear in logs, metrics or
   release artifacts.

---

### User Story 2 - Reproduce and Recover the Three-Node Deployment (Priority: P1)

A second operator installs, starts, inspects, restarts, upgrades and rolls back
the candidate using only the runbook and versioned release bundle.

**Independent Test**: From clean supported hosts, perform two installs, a
matched canary, one provider restart, N-to-N+1 activation and N+1-to-N rollback.

**Acceptance Scenarios**:

1. Doctor rejects incompatible driver, backend, identity, route, artifact or
   writable-directory state before service start.
2. Every role reaches ready without source edits or undocumented commands.
3. Provider restart changes boot identity, rejects old KV/attempt state and
   recovers through the declared same-three-node fallback when compatible.
4. Rollback preserves authoritative Repo state and discards incompatible local
   caches.

---

### User Story 3 - Sustain the Physical Pilot (Priority: P2)

An evaluator can decide whether the fixed physical deployment is stable and
serviceable from immutable canary and soak evidence rather than estimates.

**Independent Test**: Run matched single-node/distributed canaries and one
prespecified 24-hour 1 RPS soak with one scheduled provider restart.

**Acceptance Scenarios**:

1. All selected providers report matching real CUDA execution evidence and
   measured physical GPU telemetry no older than two seconds at plan commit.
2. The soak retains every submitted request and failure with no replacement run.
3. Missing mandatory evidence mechanically blocks production release.

### Edge Cases

- CUDA device UUID or driver/runtime version differs from the release manifest.
- A provider has enough configured memory but insufficient measured free memory.
- Cross-host clock skew makes telemetry appear future-dated or stale.
- NFD routes exist at startup but one inter-node face later fails.
- A provider restarts while old dependency segments remain reachable.
- Rollback encounters a newer disposable cache schema.
- The soak is interrupted by host safety limits or external power/network loss.
- One of the three physical nodes is unavailable before the campaign starts.

## Functional Requirements

- **FR-001**: Spec 106 MUST consume a Spec 105 candidate manifest whose
  `minindnCandidateOverall` is PASS and whose source, release, profile, plan,
  model and artifact digests are immutable.
- **FR-002**: Any change to algorithm, workload, artifact, retry, deadline or
  acceptance threshold MUST create a new Spec 105 candidate rather than mutate
  the physical campaign.
- **FR-003**: The physical profile MUST use exactly three declared GPU nodes and
  the same primary/fallback stage layout accepted by Spec 105.
- **FR-004**: Every production role MUST use a real identity, certificate
  validation, controller-authorized encrypted permissions, NAC-ABE attributes,
  one-time tokens, replay protection and provider permission checks.
- **FR-005**: Dummy keychains, ValidatorNull, forced authorization and plaintext
  permission responses MUST be rejected from every production cell.
- **FR-006**: Provider execution evidence MUST bind real CUDA backend, physical
  device UUID, runtime versions, model, plan, artifacts, roles and boot identity.
- **FR-007**: Placement MUST use measured physical GPU and runtime telemetry;
  configured-only capacity MUST not satisfy a production feasibility predicate.
- **FR-008**: Doctor MUST fail before startup on incompatible host, NFD routing,
  identity, trust, backend, device, artifact, plan, permission or filesystem state.
- **FR-009**: The release bundle and systemd profile MUST be installed twice from
  clean hosts without source edits or undocumented commands.
- **FR-010**: Restart, same-three-node fallback, upgrade and rollback MUST retain
  exact attempt authority and invalidate incompatible boot/KV/cache state.
- **FR-011**: Rollback MUST preserve authoritative Repo/catalog state and retain
  the failed release evidence.
- **FR-012**: Physical campaigns MUST use frozen commands, unique directories,
  no silent retry/replacement run and complete failure retention.
- **FR-013**: The 24-hour soak MUST run at 1 RPS with INFO metrics, TRACE disabled
  and one prespecified provider restart.
- **FR-014**: Only Spec 106 may change `physicalProductionOverall` from DEFERRED;
  any missing or failed mandatory dimension MUST produce BLOCK.

## Key Entities

- **Candidate Manifest**: Immutable Spec 105 release and evidence identity.
- **Physical Cluster Profile**: Three hosts, identities, routes, devices, stages
  and same-host fallback roles.
- **Physical Acceptance Record**: Commands, environment, hardware facts, all
  outcomes and dimension verdicts for one canary/drill/soak cell.
- **Production Release Gate**: Candidate plus physical security, telemetry,
  operations and soak evidence with mechanical PASS/BLOCK precedence.

## Success Criteria

- **SC-001**: 100% of production roles use real identities and matching real CUDA
  evidence; zero dummy-keychain or configured-only facts pass production gates.
- **SC-002**: Positive security bootstrap and inference pass, while 100% of
  prespecified forged, replayed, misbound and unauthorized cells fail closed.
- **SC-003**: Two clean-host installations reach ready without source edits or
  undocumented commands.
- **SC-004**: Measured telemetry used at plan commit is at most two seconds old
  and bound to the selected physical device and provider boot.
- **SC-005**: Restart, upgrade and rollback drills complete twice without
  authoritative Repo loss, duplicate terminal authority or trusted stale cache.
- **SC-006**: The 24-hour 1 RPS soak has zero incorrect outputs, zero security
  bypasses, zero unbounded resource growth and at least 99% completion.
- **SC-007**: Distributed physical p95 is no more than 2.0x the matched physical
  single-node p95, with stage and resource decomposition for at least 99% of
  completed requests.
- **SC-008**: `physicalProductionOverall=PASS` occurs only when all candidate,
  security, correctness, performance, recovery and operations artifacts exist
  and pass; otherwise it is BLOCK.

## Assumptions

- Three compatible physical GPU nodes and a second operator are not currently
  available; all Spec 106 tasks remain deferred until both are available.
- Spec 105 is complete and its candidate release remains reproducible.
- Hardware safety limits override campaign continuation and produce retained
  BLOCK evidence rather than an automatic retry.

## Explicit Non-Goals

- Algorithm, planner, wire-protocol, retry or timeout redesign.
- More than three GPU nodes, Kubernetes, autoscaling or multi-tenant service.
- Upgrading a MiniNDN result into physical evidence.
- Passing production release with missing hardware or unexecuted soak evidence.
