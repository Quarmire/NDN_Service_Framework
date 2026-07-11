# Data Model

## PermissionKind

```text
User     = tlv::UserPermission
Provider = tlv::ProviderPermission
```

Unknown values are rejected at PermissionResponse validation.

## ServiceAuthorizationRecord

```text
providerServiceName : canonical full Name URI
serviceName         : canonical unified service Name URI
permissionKind      : User | Provider
policyEpoch         : positive integer
```

Invariants:

- `providerServiceName` equals provider identity plus `serviceName`.
- `serviceName` is non-empty and contains no V1 split metadata.
- `policyEpoch > 0`.
- Invocation tokens are never stored in this record.

## ServiceAuthorizationTable

```text
map<providerServiceName, ServiceAuthorizationRecord>
```

Operations:

- `upsert(record)`: reject invalid records and lower-epoch replacement.
- `contains(providerServiceName, serviceName, kind)`: exact match only.
- `find(providerServiceName)`: immutable optional snapshot.
- `replace(kind, epoch, records)`: atomically replace records for one role.
- `snapshot()`: deterministic vector for diagnostics/tests.

All operations hold an internal mutex. Callers receive value copies, not
references into the table.
