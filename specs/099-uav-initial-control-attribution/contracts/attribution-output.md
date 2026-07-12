# Attribution Output Contract

Each run adds `initialControlAttribution` with `earliestBoundary`,
`evidenceComplete`, `telemetryAttempts`, `armAttempt`,
`automationArmTerminal`, `observerMismatch`, and `unknownReasons`.

Targeted attempts expose only provider, service, request ID, phase, status, and
timing. Payloads, tokens, certificates, credentials, and key material are
prohibited. `observerMismatch` requires a matching earlier command/Targeted
terminal followed by a conflicting later automation terminal. Sender timeout
never implies request loss, response loss, provider receipt, or execution.
