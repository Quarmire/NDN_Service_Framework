# Data Model: Repo Packet Consumer Contract

## PacketBackedManifest

- `objectName`: logical catalog identity.
- `segmentCount`: expected packet count.
- `packetNames`: non-empty ordered unique exact Data names.
- `replicaNodes`: candidate Repos serving the packet set.
- `sha256` and `size`: application object metadata; they do not replace packet
  wire validation.

Validation:

- `len(packetNames) == segmentCount`.
- Every packet name is unique.
- Every fetched packet decodes and its complete name equals the indexed name.

## OpaqueObjectManifest

- Same logical object metadata, but `packetNames` is empty.
- Payload is retrieved through object storage/reassembly.
- Internal transport segments are not application-visible packet identity.

## ExactPacketSet

- Ordered sequence of decoded Data packet records.
- Complete only when every manifest entry is present and validated.
- No partial success state.
