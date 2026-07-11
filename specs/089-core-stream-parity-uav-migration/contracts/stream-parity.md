# Stream Parity Contract

For each input vector, C++ and Python must agree on emitted sequence numbers,
next sequence, missing ranges, pending count/bytes, metrics, and adaptive fetch
decision. Session identity is `(streamId, sessionEpoch)`; old sessions are stale.
Duplicate delivery is idempotently dropped. Overflow follows one deterministic
oldest-pending rule and increments an explicit counter.

Static files, model artifacts, catalog snapshots and finite planned tensor
bundles are forbidden StreamChunk uses and must use exact-name segmented Data.
