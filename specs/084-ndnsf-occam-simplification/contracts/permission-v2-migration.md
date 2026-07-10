# V2 Permission Migration Contract

`UserPermissionTable` is not classified as wholly legacy. Current V2 user and
provider authorization paths use it for provider/service permission presence.
Only its split-name and deprecated-token representation is legacy.

The target table is a service-authorization table keyed by the canonical full
provider/service name. Each record contains:

```text
providerServiceName
serviceName
permissionKind
policyEpoch
```

It contains no invocation token. One-time UserToken and ProviderToken handling
remains independent of this table.

Migration order:

1. Inventory every insert/query/search caller and callback registration.
2. Add target-table tests for current encrypted PermissionResponse processing,
   user authorization, provider role authorization, and policy epochs.
3. Add the target table and migrate current V2 producers/consumers.
4. Verify `searchByFunctionName`, split FunctionName parsing, token-name
   decryption callbacks, and old utilities have no current V2 callers.
5. Remove only individually verified-dead symbols.
6. Run all permission, NAC-ABE, token, replay, bootstrap, normal, Targeted, and
   collaboration regressions before deleting the old table.

An old callback referenced by a registered production callback remains live
even if normal examples do not trigger it. It may be removed only after its
registration path is removed or migrated and the caller gate is empty.
