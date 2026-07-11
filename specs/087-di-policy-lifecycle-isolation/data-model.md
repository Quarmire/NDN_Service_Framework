# Data Model

## RetryReason

Typed values: `TIMEOUT`, `LEASE_REJECTED`, `LEASE_EXPIRED`, `PROVIDER_BUSY`,
`OVERLOADED`, `NON_RETRYABLE`, and `UNKNOWN`.

## RetryDecision

Contains `idempotent`, `reason`, current attempt, maximum attempts, and the
resulting `shouldRetry` decision. Diagnostic text is carried separately.

## ExperimentalPolicy

Contains explicit `enabled` state. When disabled it is identity behavior and
cannot alter assignment, authorization, response payload, or cache result.
