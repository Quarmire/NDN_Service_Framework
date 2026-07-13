# Research Decisions: NDNSF-DI MiniNDN Gate Recovery

## Decision 1: Freeze rather than revise Spec 105

**Decision**: Pin Spec 105 commit and four artifact digests in an independent
lineage lock. All new identities and result roots use `spec107-c1`.

**Rationale**: Spec 105 is a completed negative experiment. Editing it would
destroy provenance and invite selective reporting.

**Alternatives considered**:

- Add a Spec 105 R3 campaign: rejected because the user explicitly requested an
  independent revision and the T062 repetition set is frozen.
- Copy Spec 105 evidence into Spec 107: rejected; digest references preserve
  lineage without producing a second source of truth.

## Decision 2: Use one generation-scoped collaboration session

**Decision**: Keep one normal collaboration selection and execution attempt for
the complete 32-token generation; iterate stage work by token epoch inside the
same DI session.

**Rationale**: Retained Spec 105 logs show per-token request time around
0.4–1.5 seconds versus about 0.1 seconds of three-stage compute, repeated for 32
token steps. The existing code already early-closes selection when role coverage
is complete, so merely shortening `ackTimeoutMs` is neither necessary nor an
honest explanation. A generation session removes repeated setup without
changing the model, deadline, load, security path, or Core wire protocol.

**Alternatives considered**:

- Lower ACK timeout: rejected because early role coverage already exists and a
  configuration change would not remove per-token session setup.
- Use Targeted for each stage request: rejected as the primary design because
  collaboration roles, dependencies, and final authority would be duplicated in
  a second orchestration path. Existing Targeted remains available for lease and
  known-provider helper calls.
- Combine model stages or reduce output length: rejected because it changes the
  frozen product workload.
- Optimize ONNX compute first: rejected by preliminary evidence; it remains a
  falsification alternative if reconciled tracing disproves orchestration dominance.

## Decision 3: Make attribution a hard implementation gate

**Decision**: Require at least 99% timing coverage, reconciliation within 5% or
10 ms, and one avoidable component of at least 25% before implementing the
generation-session branch.

**Rationale**: This prevents a multi-optimization bundle whose benefit cannot be
attributed and prevents another post-hoc campaign design.

**Alternatives considered**:

- Implement from preliminary logs alone: rejected; current INFO timing is not a
  complete non-overlapping critical path.
- Explore multiple branches in acceptance runs: rejected; diagnostic and
  acceptance identities must remain separate.

## Decision 4: Separate production and fault executables

**Decision**: Add an experiment-only provider executable linked to the same DI
runtime for live data-path faults. Keep all fault flags out of the production
provider executable.

**Rationale**: Missing-segment and digest-corruption evidence must traverse the
live MiniNDN path, but a production debug bypass would create a security and
operational hazard.

**Alternatives considered**:

- Contract-only fault JSON: rejected because Spec 105 already proved that level
  and recorded `networkInjection=false`.
- Environment-variable fault hooks in production provider: rejected because
  accidental activation and release-bundle ambiguity are avoidable.
- Manipulate host/default NFD: rejected; MiniNDN-owned processes and interfaces
  are the only permitted targets.

## Decision 5: Reuse one content-addressed artifact set

**Decision**: Materialize and hash the three ONNX files once, remove `.pt`
intermediates, make the set read-only, and reference it from every cell.

**Rationale**: Spec 105 lost two cells to disk exhaustion caused by repeated
large exports. Artifact reuse removes the known infrastructure failure without
altering compute.

**Alternatives considered**:

- Copy artifacts into every result directory: rejected due disk amplification.
- Delete all binaries after each run: rejected because regeneration adds time,
  disk risk, and identity ambiguity.

## Decision 6: Local process supervision is not physical systemd evidence

**Decision**: Execute exact packaged commands under an isolated local process
supervisor and retain static unit hardening checks. Label the evidence
`local-process-supervision`.

**Rationale**: The available environment can prove real process lifecycle and
rollback but cannot prove production PID-1 behavior. Spec 106 owns that claim.

**Alternatives considered**:

- Replace `ExecStart` with `/bin/true`: rejected for operations acceptance; it
  remains only a static syntax/hardening check.
- Claim local supervisor equals systemd: rejected as evidence inflation.

## Decision 7: Run recovery independently; gate soak on everything

**Decision**: Live fault cells may execute after correctness and safety tests
even if performance later fails. Canary and package lifecycle may close their
own evidence. The 24-hour soak starts only after all predecessor dimensions pass.

**Rationale**: Independent evidence avoids repeating Spec 105's situation where
one failed performance gate prevented learning about live recovery, while the
expensive soak remains protected by strict preconditions.

## Decision 8: Threshold verdicts are per repetition

**Decision**: Each 60-second repetition must meet every threshold independently.
Bootstrap intervals and aggregate summaries are descriptive only.

**Rationale**: Pooling can hide one unstable or failed cell and weakens the
frozen acceptance contract.
