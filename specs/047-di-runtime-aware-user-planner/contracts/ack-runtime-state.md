# Contract: Runtime-Aware ACK Payload

## Purpose

Allow a provider ACK to carry enough runtime information for a user-side planner
to make a current assignment decision without making the provider responsible
for global planning.

## Logical Shape

```json
{
  "schema": "ndnsf-di-runtime-ack-v1",
  "providerRuntimeState": {
    "providerName": "/provider/A",
    "timestampMs": 0,
    "activeRoleCount": 0,
    "queueLength": 0,
    "estimatedQueueWaitMs": 0,
    "freeGpuMemoryMb": 0,
    "freeCpuMemoryMb": 0,
    "supportedBackends": ["onnx-cpu", "onnx-cuda"],
    "fragmentStates": [],
    "peerMetrics": [],
    "confidence": 1.0
  },
  "leaseOffers": [],
  "metricDigest": "",
  "notes": ""
}
```

## Compatibility

- Existing ACK status, message, payload, UserToken, and ProviderToken behavior
  must remain valid.
- If runtime-aware mode is optional, missing payload fields fall back to
  conservative scoring.
- If runtime-aware mode is required, missing fields produce a structured
  unsupported-feature reason.

## Required Validation

- Provider name in runtime state must match the ACK provider.
- Lease offers must bind the same request id as the ACK.
- Fragment keys in lease offers must match the role being offered.
