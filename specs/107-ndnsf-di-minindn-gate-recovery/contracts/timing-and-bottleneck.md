# Timing and Bottleneck Contract

Timing event components:

```text
admission ack-selection plan-lease queue compute encode-decode
dependency-fetch dependency-publish response inter-token
```

Every event carries candidate, campaign, generation, token epoch, request,
attempt, provider, boot, role, monotonic start/end, status, and sampler decision.
Payload/token/tensor values are forbidden.

The analyzer rejects negative/overlapping spans, identity mismatch, missing
components, coverage below 99%, or unexplained time exceeding 5% and 10 ms.

`bottleneck-decision.json` selects exactly one branch when its avoidable time is
the largest and at least 25% of warm token-step time. Otherwise its verdict is
`BLOCK_REPLAN`. Diagnostic evidence is never release-eligible.
