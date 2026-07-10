# Contract: Repo Packet Consumer API

## Store

```text
put_signed_packets(logicalObjectName, packets, object metadata) -> Manifest
```

Preconditions:

- `packets` contains application-produced signed NDN Data wires.
- Packet names remain under the declared original Data prefix.

Postconditions:

- `Manifest.packetNames` contains every exact complete Data name in order.
- Repo does not rename, re-segment, or re-sign packet wires.

## Retrieve

```text
get_signed_packets(Manifest) -> ordered ExactPacketSet
```

Preconditions:

- `Manifest.packetNames` is non-empty, unique, and count-consistent.

Postconditions:

- One packet is returned for every indexed name, in the same order.
- Every returned complete name equals the requested manifest name.
- Any missing or mismatched packet fails the whole operation.

## Object and Packet Views

```text
put/get object bytes     -> manifests without packetNames
put/get signed packets   -> manifests with packetNames
```

For a packet-backed manifest, `get_signed_packets` returns exact original wires
and `get` may return reassembled application content. Neither read view changes
the stored packet representation.
