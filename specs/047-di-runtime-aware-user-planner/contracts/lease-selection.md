# Contract: Generic Lease-Aware Selection

## Purpose

Ensure a provider executes a role only when the user selected a lease that the
provider actually granted and still considers valid. NDNSF core validates the
generic lease; NDNSF-DI validates the DI resource binding payload.

Admission lease validation is opt-in. A service that does not enable lease
validation must continue to accept the current NDNSF selection flow, subject to
the existing ProviderToken, UserToken, NAC-ABE, provider permission, and replay
checks.

## Selection Entry Shape

```json
{
  "schema": "ndnsf-lease-selection-v1",
  "requestId": "<request-id>",
  "serviceName": "/Inference/NativeTracer",
  "providerName": "/provider/A",
  "leaseId": "<lease-id>",
  "resourceBindingSchema": "ndnsf-di-lease-binding-v1",
  "resourceBinding": {
    "roleId": "/Stage/0",
    "fragmentKey": {
      "modelId": "qwen-tiny",
      "modelDigest": "sha256:...",
      "runtimeBackend": "onnx-cuda",
      "precision": "fp16",
      "splitStrategy": "pipeline",
      "stageIndex": 0,
      "stageCount": 3,
      "layerStart": 0,
      "layerEnd": 7,
      "shardIndex": 0,
      "shardCount": 1,
      "fragmentDigest": "sha256:..."
    }
  }
}
```

## Provider Validation Rules

The provider must reject without executing when:

- lease id is unknown;
- lease is expired;
- lease was already consumed;
- request id does not match;
- service name does not match;
- generic resource binding proof does not match;
- reserved resource is no longer available;
- provider token validation fails.

NDNSF-DI additionally rejects when role id or fragment key does not match the
DI lease binding.

These lease checks run after the existing security checks are available and do
not replace them.

## Rejection Result

Rejected selections must expose a structured reason so the user can replan:

```json
{
  "status": false,
  "reasonCode": "LEASE_EXPIRED",
  "leaseId": "<lease-id>",
  "serviceName": "/Inference/NativeTracer"
}
```
