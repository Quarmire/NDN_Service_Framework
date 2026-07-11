# Data Model

## ProviderCapabilityHint v2

The existing typed envelope remains the authority. V2 requires an explicit
known schema and preserves provider/service identity, readiness, drain state,
reason, message, runtime hint, peer network metrics, lease offers, operation
status, and versioned service payload.

## AckCompatibilityMode

- `typed-only`: typed v2/v1 is required; no flat fallback.
- `mixed`: typed v2/v1 is preferred; flat legacy is accepted only when the
  typed field is absent.

## AckCompatibilityCounters

Counters: `typed`, `legacy`, `matchingDual`, `conflictingDual`, `malformedTyped`,
`unknownTypedVersion`, and per-field conflicts.

## AckDecodeResult

Contains the authoritative `ProviderCapabilityHint`, source (`typed` or
`legacy`), parsed transport fields, conflicting field names, and a snapshot of
compatibility counters.
