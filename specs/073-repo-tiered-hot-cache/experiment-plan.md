## Material Passport

- **Schema**: ARS-9
- **Mode**: experiment-agent / plan
- **Material type**: code experiment plan
- **Source**: Spec 073 implementation and local MiniNDN environment
- **Verification target**: deterministic functional reproduction

# MiniNDN Experiment Plan: Tiered Repo Authority and Cache

## Research Question

Does a real NDNSF network Repo preserve objects through restart while enforcing a bounded hot cache that exhibits observable cold misses, repeated-read hits, LRU eviction, and SQLite fallback?

## Hypotheses

- **H1**: An object acknowledged before Repo restart remains byte-identical afterward.
- **H2**: The first post-restart fetch increments miss/backing-read counters; an immediate repeat increments hit without another backing read.
- **H3**: A controlled working set larger than the cache budget causes eviction while `usedBytes <= budgetBytes`.
- **H4**: Fetching an evicted object still succeeds from SQLite and increments miss/backing-read counters.

## Variables

- **Independent variables**: access order, object size, cache budget, process restart.
- **Dependent variables**: returned digest, hit/miss/admission/eviction counters, backing reads/writes, used bytes, exit status.
- **Controls**: topology, Repo identity, database path, payload generator, NDNSF policy, NFD/NLSR setup, timeout values.
- **Confounds controlled**: NDN Content Store is not used as evidence; cache status counters from the application Repo are the source of cache-path classification.

## Procedure

1. Start the existing generic Repo MiniNDN topology with Repo A configured for a small cache and persistent SQLite path.
2. Store deterministic objects on Repo A and write their names/digests to a state file.
3. Query and record `CACHE_STATUS`.
4. Stop Repo A cleanly and restart it with the same SQLite path.
5. Confirm the test objects are absent from process-local memory by the restarted counter baseline/access behavior.
6. Fetch object A once and record a miss/backing read.
7. Fetch object A again and record a hit with byte-identical payload.
8. Fetch enough additional deterministic objects to exceed the budget and observe at least one eviction.
9. Fetch an evicted object and confirm SQLite fallback and digest integrity.
10. Write a JSON summary and fail the process if any invariant is false.

## Acceptance Thresholds

- All expected payload SHA-256 values match exactly.
- `usedBytes <= budgetBytes` at every sampled status.
- Post-restart sequence contains at least one miss, one hit, one backing read, and one eviction.
- The final fetch of an evicted object succeeds.
- No statistical inference is needed because the test is deterministic; any mismatch is a functional failure.

## Reproducibility Record

The result must preserve the exact command, topology, output directory, SQLite path, cache budget, logs, and JSON summary. A rerun passes only if the same logical assertions hold; exact timing equality is not required.

## Verified Result

The 2026-07-10 MiniNDN run used `Experiments/Topology/AI_Lab.conf`, an 8192-byte
cache, three 4096-byte objects, the same Repo A SQLite directory across restart,
and access order `A,A,B,C,A`. The final status reported one hit, four misses,
four backing reads, four evictions, 6200 used bytes, and every acceptance check
true. Evidence is in
`results/distributed_repo_tiered_cache_minindn/tiered-cache-summary.json`.
