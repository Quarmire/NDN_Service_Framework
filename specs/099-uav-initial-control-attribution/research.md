# Research

## Correct Evidence Before Network Treatment

**Decision**: Add named pending/timeout command-state factories.

**Rationale**: Existing aggregate construction puts attempt epoch in `rttMs`
and timeout budget in `updatedMs`; timeout construction puts terminal epoch in
`rttMs` and zero in `updatedMs`. Automation then misses a real Arm timeout.

**Alternatives considered**: Relaxing stale-state checks or extending deadlines
would hide the defect and amount to unsafe evidence acceptance or timeout tuning.

## Reuse Sender-Side Targeted Phases

**Decision**: Correlate queued/dispatched/response/timeout/rejected phases by
request ID and service.

**Rationale**: Existing records provide the strongest Ground Station evidence
without sensitive logging or transport changes.

**Alternatives considered**: Sender timeout cannot prove request-versus-response
loss. Broad packet capture/provider payload logs would expand and perturb scope.

## Preserve Two Distinct Baselines

**Decision**: Run 03 is the command observer baseline; run 04 is the initial
telemetry/deadline-overlap baseline.

**Rationale**: Run 03 has Arm Targeted timeout at 10.5 s and automation expiry
at 12.0 s. Run 04 has initial telemetry timeout, replacement dispatch only 25 ms
before wait expiry, and a response afterward.

## Diagnostic, Not Reliability Treatment

**Decision**: Judge the frozen cell by attribution integrity, not completion.

**Rationale**: Delivery policy is unchanged. Retries, wait extension, polling
changes, and Targeted timeout changes remain deferred.
