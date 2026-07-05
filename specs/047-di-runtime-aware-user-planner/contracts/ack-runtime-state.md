# Contract: Runtime-Aware ACK Metadata

## Purpose

Allow a provider ACK to carry enough runtime information for a user-side planner
to make a current assignment decision without making the provider responsible
for global planning. The outer envelope is reusable NDNSF core metadata; the DI
payload is service-defined.

## Logical Shape

```json
{
  "schema": "ndnsf-ack-metadata-v1",
  "providerRuntimeHint": {
    "providerName": "/provider/A",
    "timestampMs": 0,
    "activeRoleCount": 0,
    "queueLength": 0,
    "estimatedQueueWaitMs": 0,
    "capacityHints": {},
    "peerMetrics": [],
    "confidence": 1.0
  },
  "leaseOffers": [],
  "servicePayloadSchema": "ndnsf-di-runtime-ack-v1",
  "servicePayload": {
    "fragmentStates": [],
    "diCapacityHints": {},
    "kvCacheHints": []
  },
  "metricDigest": "",
  "notes": ""
}
```

## Compatibility

- Existing ACK status, message, payload, UserToken, and ProviderToken behavior
  must remain valid.
- Services that do not opt into admission leases may ignore `leaseOffers`.
- The absence of `leaseOffers` must not change legacy behavior unless the user
  request or service configuration explicitly requires lease-aware admission.
- If runtime-aware mode is optional, missing payload fields fall back to
  conservative scoring.
- If runtime-aware mode is required, missing fields produce a structured
  unsupported-feature reason.
- NDNSF core validates only the generic envelope and lease envelope. NDNSF-DI
  validates `servicePayload`.
- ProviderToken remains the security/selection proof. Admission lease metadata
  is only an admission/resource proof.

## Required Validation

- Provider name in runtime state must match the ACK provider.
- Lease offers must bind the same request id as the ACK.
- DI resource bindings inside lease offers must match the role being offered.
