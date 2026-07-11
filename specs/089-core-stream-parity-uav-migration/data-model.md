# Data Model

- `StreamInfo`: stream/session identity and fetch defaults.
- `StreamChunk`: sequence, deadline, frame/segment metadata, opaque payload and
  codec-neutral FEC description.
- `StreamMetrics`: produced/received/emitted/duplicate/stale/gap/timeout/NACK,
  bytes and overflow counters.
- `StreamFetchDecision`: bounded window/lookahead/lifetime/missing timeout.
- `StreamHealth`: presentation snapshot; it does not decide codec behavior.
