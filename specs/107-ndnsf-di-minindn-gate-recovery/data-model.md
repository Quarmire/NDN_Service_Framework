# Data Model: NDNSF-DI MiniNDN Gate Recovery

## LineageLock

Fields:

- `schema`: `ndnsf-di-lineage-lock-v1`
- `predecessorSpec`: fixed Spec 105 path
- `frozenCommit`: full Git commit
- `files[]`: path, SHA-256, classification
- `predecessorReleaseId`
- `predecessorMiniNdnVerdict`: `BLOCK`
- `predecessorPhysicalVerdict`: `DEFERRED`

Validation: every path is repository-relative, exists, and has the exact digest;
no duplicate path; no write operation is exposed.

## CandidateIdentity

Fields:

- namespace `spec107-c1`
- source, lineage, workload, profile, model, tokenizer, plan, artifact-set,
  trust-policy, and command digests
- generated full candidate ID
- canonical complete-candidate digest, embedded in every campaign identity
- creation timestamp and generator version

State: `DRAFT -> FROZEN -> EXECUTED -> GATED`. Any input digest change creates a
new candidate namespace; it cannot mutate a frozen identity.

Only the `FROZEN` state is executable. Candidate manifests use an exact schema,
UTC second-resolution creation time, and non-empty generator version; unknown
or missing top-level fields are invalid. Candidate and campaign manifest roots
must be JSON objects; scalar and array roots are invalid.

The readable candidate ID uses six digest prefixes. Campaign authorization also
requires the complete canonical candidate digest, so fields not displayed in
that ID cannot be changed while retaining a campaign identity.

Campaign validation recomputes its canonical ID from every binding field and
derives eligibility from campaign kind. Stored IDs and eligibility labels are
evidence to verify, never authority by themselves. Campaign manifests also use
an exact top-level schema; missing or unknown fields fail before any binding is
accepted.

## TimingSpan and TimingDecomposition

`TimingSpan` carries request/generation/token IDs, provider/role, attempt and
boot identities, component enum, monotonic start/end, sampled flag, and status.

`TimingDecomposition` carries ordered spans, observed end-to-end time,
reconciled time, unexplained time/ratio, coverage ratio, and overlap/gap errors.
Valid components are admission, ACK/selection, plan/lease, queue, compute,
encode/decode, dependency fetch/publish, response, and inter-token.

## BottleneckDecision

Fields:

- candidate and diagnostic campaign identity
- hypotheses with measured absolute/relative time
- selected branch and source touchpoints
- dominance percentage
- reconciliation and coverage statistics
- falsification condition
- rejected alternatives
- verdict: `SELECTED | BLOCK_REPLAN`

Exactly one branch may be selected and it must have dominance >=25%.

## QwenGenerationSessionSpec

Fields:

- schema and candidate/plan/model/artifact digests
- logical session and execution attempt epoch
- service and ordered roles/providers
- prompt/context reference, input length, max generated tokens
- token epoch, KV security/boot bindings, deadline
- dependency/feedback topics and final-response role

State:

```text
CREATED -> SELECTING -> ACTIVE(token 0..31)
ACTIVE -> REBUILDING (one replacement/full-context rebuild)
ACTIVE|REBUILDING -> COMPLETED | TERMINAL | CANCELLED
```

Only one transition to `COMPLETED` or `TERMINAL` is authoritative.

## ArtifactSet

Fields: artifact-set digest, model revision, three role/path/size/digest rows,
filesystem device/inode identity, read-only status, export provenance, and
retention classification. `.pt` files are invalid after materialization.

## CampaignPreflight

Fields:

- campaign kind and ID
- candidate/command/profile/artifact digests
- unique output path and writer/ownership checks
- free/projected/reserve bytes
- required predecessor gates
- host/backend/MiniNDN facts
- verdict `PASS | INVALID_PREFLIGHT`
- stable reasons

State is immutable after any role starts.

## OwnedProcess

Fields: PID, process group, `/proc` start time, parent PID, campaign ID, role,
provider identity, boot identity, command digest, executable digest, start/end
times, and cleanup status.

Destructive actions require an exact current match of all ownership fields.

## LiveFaultCell

Fields:

- cell ID/type, target `OwnedProcess` or data object
- trigger condition and monotonic injection time
- `networkInjection=true`
- intended and observed effect
- before/after provider boot and attempt epoch
- cancel/supersede evidence
- result/terminal reason and authority count
- bounded-resource snapshots
- cleanup proof
- verdict `PASS | BLOCK | INVALID`

## LocalOperationsRecord

Fields: staging root, release/plan/evidence identities, supervisor class,
commands and exit codes, role readiness, structured status/metrics digests,
restart/upgrade/rollback events, Repo before/after digest, cache decision,
process cleanup, and verdict.

## SuccessorReleaseGate

Fields: candidate and lineage identities; evidence manifest; correctness,
performance, recovery, application-security, and local-operations dimensions;
`minindnCandidateOverall`; fixed `physicalProductionOverall=DEFERRED`;
limitations; source and generator commits.

Any missing, malformed, digest-mismatched, or BLOCK local dimension yields
overall BLOCK.
