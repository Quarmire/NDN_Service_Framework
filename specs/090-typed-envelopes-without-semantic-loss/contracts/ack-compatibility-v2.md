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

The default reader is `typed-only`. Operators may temporarily enable the
bounded legacy reader with `NDNSF_ACK_COMPATIBILITY_MODE=mixed` or the decoder
API's `mode="mixed"`. This reader expires at the next major release or
2026-12-31, whichever comes first. It may be removed only after current
producer scans remain clean, typed-only acceptance reports zero legacy use,
and a mixed-reader campaign reports zero unexplained conflicts.
