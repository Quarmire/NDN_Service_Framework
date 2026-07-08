# Runtime Envelope Contract

## Provider Capability ACK Field

ACK payloads may include:

```text
providerCapabilityHint=json64:<stable-json>
```

The decoded JSON uses:

```json
{
  "schema": "ndnsf-provider-capability-v1",
  "providerName": "/provider/A",
  "serviceName": "/service/name",
  "ready": true,
  "drainState": "ACTIVE",
  "reasonCode": "",
  "message": "ready",
  "runtimeHint": {
    "providerName": "/provider/A",
    "queueLength": 0,
    "estimatedQueueWaitMs": 0,
    "freeMemoryMb": 0,
    "freeGpuMemoryMb": 0
  },
  "servicePayloadSchema": "app-schema-v1",
  "servicePayload": {}
}
```

Apps may add service-specific details only inside `servicePayload`.

## Operation Status Field

ACK or response payloads may include:

```text
operationStatus=json64:<stable-json>
```

The decoded JSON uses the shared lifecycle states:

```json
{
  "operationId": "op-1",
  "operation": "STORE",
  "serviceName": "/NDNSF/DistributedRepo",
  "providerName": "/repo/A",
  "state": "RUNNING",
  "reasonCode": "",
  "message": "fetching",
  "progress": 0.5
}
```

## Stream vs Large-Data Contract

- Use stream helpers for continuous or near-live publication: video, telemetry,
  live logs, sensor feeds.
- Use exact-name large-data helpers for files, recordings, manifests, model
  artifacts, catalog snapshots, and DI tensor bundles.

