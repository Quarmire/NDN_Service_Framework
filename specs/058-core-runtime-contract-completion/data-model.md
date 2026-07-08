# Data Model: Core Runtime Contract Completion

## ProviderCapabilityHint

- `providerName`: provider identity name.
- `serviceName`: service the hint applies to.
- `ready`: whether new requests are acceptable.
- `drainState`: optional state such as `ACTIVE`, `DRAINING`, `UNAVAILABLE`, or `MAINTENANCE`.
- `reasonCode`: structured reason when not ready or draining.
- `message`: human-readable status.
- `runtimeHint`: generic runtime queue/memory/peer metrics.
- `leaseOffers`: optional generic admission leases.
- `operationStatus`: optional ServiceOperationStatus.
- `servicePayloadSchema`: app-specific payload schema name.
- `servicePayload`: opaque app-specific payload.

## ServiceOperationStatus

- `operationId`: stable operation id.
- `operation`: app-visible operation name.
- `serviceName`: service name.
- `providerName`: provider identity.
- `requestId`: related request id when available.
- `state`: `QUEUED`, `RUNNING`, `WAITING_INPUT`, `DONE`, `FAILED`, `CANCELED`, `EXPIRED`.
- `reasonCode`: structured failure/wait reason.
- `message`: human-readable status.
- `progress`: 0.0 to 1.0.
- `resultReference`: optional DataProductReference-like object.
- `retryAfterMs`, `createdAtMs`, `updatedAtMs`, `expiresAtMs`.

## DataProductReference

- `name`: exact NDN Data prefix or object name.
- `producerName`: producer identity.
- `serviceName`: service that produced it.
- `objectClass`: artifact, recording, catalog, tensor-bundle, etc.
- `contentType`: MIME-like content type.
- `digest`, `sizeBytes`, `segmentCount`, `freshnessMs`, `metadata`.

## ServiceDiscoverySnapshot

- `serviceName`: queried service.
- `providers`: provider records with provider identity, readiness, drain state,
  reason code, freshness, source, runtime hint, and capability payload.
- Derived categories: `ready`, `draining`, `unavailable`, `stale`.

## ProviderNetworkMatrix

Existing core entity. This feature keeps the data model and requires app code
to use it instead of duplicating pair-metric ranking.

