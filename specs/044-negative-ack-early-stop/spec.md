# Feature Specification: Negative ACK Early Stop

**Feature Branch**: local worktree
**Created**: 2026-06-30
**Status**: Draft
**Input**: Reuse existing `RequestAckMessage.status=false` as a negative ACK and add reason-code diagnostics, user-side recording, known-provider early stop, regression tests, and MiniNDN validation.

## User Scenarios

### Scenario 1 - Known providers all reject a request

When a user sends a request to an explicit provider list and every provider replies with `status=false`, the user should stop waiting for the full ACK collection window and request timeout. The request fails early with diagnostics showing which providers rejected the request and why.

### Scenario 2 - Mixed positive and negative ACKs

When some providers reject and at least one provider accepts, selection continues using only `status=true` ACKs. Negative ACKs remain visible in logs and custom-selection candidates but are never selected.

### Scenario 3 - Discovery or unknown provider count

When the request is not addressed to an explicit provider list and the user does not know the full responder set, a negative ACK is diagnostic only. The user must not fail early because another provider may still accept later.

## Requirements

- Reuse `RequestAckMessage.status=false`; do not introduce a new wire message.
- Treat `RequestAckMessage.message` as the reason code string for negative ACKs.
- Preserve `RequestAckMessage.payload` for optional provider diagnostics such as queue, rank, GPU state, or admission metrics.
- Record negative ACK providers and reason codes in the user pending-call state.
- Add trace logs for negative ACK receipt and all-negative known-provider early stop.
- Early stop only when:
  - the request has a non-empty explicit provider list,
  - every explicit provider has replied with `status=false`,
  - no positive ACK has been recorded,
  - no provider has already been selected, and
  - no response has already arrived.
- Early stop must invoke the same timeout/failure callback path used by normal request timeout.
- Built-in and custom selection must continue to ignore `status=false` ACKs for provider selection.
- Add regression coverage for known-provider all-negative ACKs.
- Validate with a MiniNDN smoke/performance run after the local regression passes.

## Non-Goals

- Do not change NAC-ABE, permission bootstrap, controller behavior, or token semantics.
- Do not add NDNSF request retry or ACK retry.
- Do not make negative ACK terminal for service discovery requests without an explicit provider list.
- Do not redesign ACK payload schemas; keep the current generic payload bytes.

## Acceptance Criteria

- Existing selective ACK custom-selection regression still passes with a mixed positive/negative provider set.
- New negative ACK regression shows early stop before a long request timeout when all known providers reject.
- User logs include the rejecting providers and reason codes.
- MiniNDN validation completes using the updated build.
