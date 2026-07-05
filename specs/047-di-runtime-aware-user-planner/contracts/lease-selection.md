# Contract: Lease-Aware Selection

## Purpose

Ensure a provider executes a role only when the user selected a lease that the
provider actually granted and still considers valid.

## Selection Entry Shape

```json
{
  "schema": "ndnsf-di-lease-selection-v1",
  "requestId": "<request-id>",
  "providerName": "/provider/A",
  "roleId": "/Stage/0",
  "leaseId": "<lease-id>",
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
```

## Provider Validation Rules

The provider must reject without executing when:

- lease id is unknown;
- lease is expired;
- lease was already consumed;
- request id does not match;
- role id does not match;
- fragment key does not match;
- reserved resource is no longer available;
- provider token validation fails.

## Rejection Result

Rejected selections must expose a structured reason so the user can replan:

```json
{
  "status": false,
  "reasonCode": "LEASE_EXPIRED",
  "leaseId": "<lease-id>",
  "roleId": "/Stage/0"
}
```
