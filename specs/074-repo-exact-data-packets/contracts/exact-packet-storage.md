# Exact Packet Storage Contract

## Store packet set

Existing operations `STORE_PACKETS`, `STORE_PACKET`, `STORE_PACKET_BATCH`, and
`STORE_PACKET_PULL` accept wire packets. For each packet:

```json
{
  "name": "/data/example/v=42/seg=0",
  "segment": 0,
  "wireSha256": "...",
  "wireB64": "..."
}
```

The service decodes `wireB64`; `name`, `segment`, and `wireSha256` are
validation assertions, not storage-key generators.

## Prepare one exact packet

```text
operation: FETCH_PACKET_PREPARE
request:   { dataName }
response:  { dataName, wireSha256, forwardingHints }
```

The response `dataName` must equal the request exactly. The caller then sends an
Interest for that name and receives the stored wire packet.

## Prepare a manifest packet set

```text
operation: FETCH_PREPARE
request:   { objectName }
response:  { dataName, versionedDataName, packetNames[], manifest, forwardingHints }
```

For packet-backed objects, `packetNames` are authoritative. `dataName` and
`versionedDataName` are derived from those names, never from the Repo node or
logical object name.

## Errors

| Code/message | Meaning |
|---|---|
| `repo-invalid-data-wire` | Submitted bytes are not valid NDN Data |
| `repo-data-name-mismatch` | Declared name differs from encoded name |
| `repo-data-wire-conflict` | Existing exact name has different wire |
| `repo-packet-set-invalid` | Packet ordering/version/final-block invariants fail |
| `repo-packet-miss` | Exact Data name is absent |
