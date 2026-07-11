# ACK Compatibility V2 Contract

Typed root field: `providerCapabilityHint`.

Known schemas:

```text
ndnsf-provider-capability-v1  reader compatibility only
ndnsf-provider-capability-v2  current producer and reader
```

Typed authority applies to provider identity, service identity, ready/drain,
reason/message, runtime queue/work/capacity/network, lease, operation status,
and service payload. In mixed mode, duplicate flat values are diagnostic only.
If typed data is present but malformed or unknown, decoding fails; it never
falls back to flat fields.
