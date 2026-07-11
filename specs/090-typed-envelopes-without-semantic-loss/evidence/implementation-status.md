# Implementation Status

**Implementation commit**: `72dc052`

`ProviderCapabilityHint` v2 is now the sole current ACK capability root for DI
Python, native DI C++, and Repo Python producers. Common consumers use the
shared fail-closed decoder. The pybind collaboration selector also decodes the
typed service payload before role assignment and capacity scoring.

The decoder provides typed-only and explicit mixed modes, typed authority,
v1 typed-reader compatibility, and process-local counters for typed, legacy,
matching dual, conflicting dual, malformed typed, unknown typed version, and
per-field conflicts. Domain fields remain in each versioned `servicePayload`;
`GenericAckMetadata`, stored Repo/DI/UAV state, exact Data wire, and security
paths are unchanged.

The first typed-only MiniNDN attempt received valid typed ACKs but timed out
because the pybind collaboration selector still read flat `roles`. The selector
was migrated to the typed root, rebuilt, and both acceptance runs then passed.

