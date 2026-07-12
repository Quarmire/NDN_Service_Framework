# Final Validation

## Material Passport
- Origin Skill: experiment-agent
- Origin Mode: validate
- Origin Date: 2026-07-12
- Verification Status: ANALYZED

The single frozen both-end TRACE cell is retained at
`results/spec104-targeted-user-lifecycle-attribution-loss05-final`. It used
control-only, five runs, 5% configured loss, a 60-second automation ceiling,
and `--targeted-lifecycle-trace`. The campaign accepted 3/5 runs and completed
the full control sequence in 3/5. There were zero lifecycle aborts and zero
duplicate automation dispatches. One final telemetry attempt was deliberately
retained as unterminated when its failed run shut down.

Six unique telemetry attempts reached an explicit sender timeout, and every one
has user-side attribution:

| Count | User boundary | Provider boundary | Interpretation |
|---:|---|---|---|
| 3 | `request-published-no-ack` | `ack-published-selection-not-completed` | Provider logged `ACK_PUBLISHED`, but user logged no `ACK_RECEIVED` before timeout. |
| 2 | `request-published-no-ack` | `request-received-decrypt-not-completed` | Provider received the request but did not complete decrypt or publish an ACK before timeout. |
| 1 | `selection-published-no-response` | `response-published-no-user-response` | User published Selection and provider published Response; user later logged `RESPONSE_SKIPPED_NO_PENDING` after timeout cleanup. |

The additional unterminated attempt has the first category pair: provider
published ACK while user had not observed ACK before shutdown. Thus the
evidence separates three boundaries rather than supporting one universal root
cause: provider request decryption, ACK publication-to-user observation, and
Response arrival after user pending-state expiry. It does not prove whether
loss, SVS fetch timing, validation/decryption scheduling, or another transport
mechanism caused those boundaries.

Verification passed: Python 28/28; C++ 219/219 with no errors; strict Spec Kit
structure audit PASS; core C++ diff empty. The trace summaries contain event
names and request IDs only, not payload, token, or key material.

Post-implementation audit verdict: **PASS**. Requirements, tasks, parser tests,
the executed MiniNDN artifact, and the reported denominators agree; no security,
migration, architecture-boundary, or unsupported-success finding remains.

## Fallacy Scan (11/11)

1. **Small sample**: five stochastic runs cannot estimate reliability.
2. **Selection bias**: this diagnostic cell was chosen after earlier failures.
3. **Regression to the mean**: 3/5 completion may vary naturally.
4. **Post hoc causality**: temporal boundaries do not prove root causes.
5. **Survivorship bias**: failed and unterminated attempts are retained.
6. **Look-elsewhere effect**: only the predeclared user/provider categories are reported.
7. **Researcher degrees of freedom**: one frozen command and no replacement run were used.
8. **Instrumentation effect**: ServiceUser and ServiceProvider TRACE perturb hot paths.
9. **Metric substitution**: lifecycle attribution is not control reliability.
10. **Denominator ambiguity**: six explicit timeouts and one unterminated attempt are reported separately.
11. **Overgeneralization**: evidence applies only to this MiniNDN 5% diagnostic cell.

Because both endpoints ran at TRACE, this cell is not performance-comparable to
prior runs and makes no performance or reliability improvement claim.

Next: instrument or inspect the SVS publication-fetch/validation boundary for
ACK and Response delivery using the exact request IDs, without changing retry,
suppression, cryptography, or timeout policy yet.
