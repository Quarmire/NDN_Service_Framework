# Final Validation

## Material Passport
- Origin Skill: experiment-agent
- Origin Mode: validate
- Origin Date: 2026-07-12
- Verification Status: ANALYZED

One TRACE-enabled five-run 5% cell ran at
`results/spec103-targeted-core-lifecycle-attribution-loss05-final`.
Control/full sequence completed 3/5, but TRACE perturbs the hot path, so this is
not performance-comparable with prior cells.

Four unique telemetry timeouts occurred:

- 2 `request-received-decrypt-not-completed`: provider core observed the request
  but did not log completed decryption before user timeout;
- 2 `ack-published-selection-not-completed`: provider decrypted the request,
  stored pending state, decided and published ACK, but never reached Targeted
  acceptance/handler execution.

No timeout was provider-unobserved, handler-returned, response-published, or
publish-failed in this cell. Thus the current boundary is pre-handler bootstrap:
request decryption and ACK→Selection progression, not response publication.

Python 27/27 and the unchanged core tree's C++ 219/219 pass. Zero lifecycle
aborts, duplicate commands, or unterminated automation occurred. Fallacy scan
11/11 flags TRACE perturbation, small selected samples, and causal overreach;
all runs/timeouts are retained and no reliability claim is made.

Next: correlate user-side request decrypt/ACK receipt/Selection publication for
these request IDs before modifying cryptography, suppression, or retry policy.
