# Catalog and Repair Contract

Catalog deltas identify `sourceRepo`, `sourceBootId`, `sinceSequence`, `sourceSequence`, membership heartbeat, and ordered entries. Entries bind object generation, digest, state, and source sequence.

Repair jobs are durable and idempotent. A worker leases a job, copies from a currently live source to an eligible target, verifies the write receipt, reports the result, and releases the lease. A later loss may create a new job for the same object and a different repair epoch.

`CATALOG_DIGEST` returns deterministic bucket hashes. `CATALOG_BUCKET` returns entries for mismatching buckets. This detects state omitted from ordinary deltas after compaction or restart.
