# Permission V2 Migration Contract

## Canonical record

Each installed authorization is represented by:

```text
providerServiceName = /<provider>/<serviceName...>
serviceName         = /<serviceName...>
permissionKind      = USER or PROVIDER
policyEpoch         = controller-issued epoch
```

The table performs exact provider-service, service, and kind matching. A lower
epoch cannot replace a newer role snapshot; the same or a newer epoch replaces
that role atomically. The deprecated PermissionEntry token remains decodable at
the wire boundary but is never indexed or used for invocation authorization.

## Authority and security

- Permissions come only from controller-signed PermissionResponse Data encrypted
  to the target identity certificate.
- User and provider responses install separate permission kinds.
- REQUEST and SELECTION remain `/SERVICE/<service>` NAC-ABE attributes.
- ACK and RESPONSE remain `/PERMISSION/<service>` NAC-ABE attributes.
- UserToken, ProviderToken, Targeted token batches, and replay rejection are
  unchanged.

## V1 removal

`PublishRequest`, split ServiceName/FunctionName helpers, Bloom-filter request
targeting, V1 parsers and fallback callbacks, and token-name permission
installation are removed. A legacy split request may share the `NDNSF/REQUEST`
marker, but its additional function/Bloom components become part of a different
V2 service name and therefore fail the exact authorization lookup before any
handler executes.

## Compatibility

This is an explicit breaking migration on the Experimental branch. No V1 wire
or API alias remains. `LegacyAckStrategyHandler` is a source-only callback
adapter and does not restore V1 naming; its later removal requires a separate
caller inventory.
