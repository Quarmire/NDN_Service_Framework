# Implementation Plan: Exact NDN Data Packet Repository

**Branch**: `Experimental` | **Date**: 2026-07-10 | **Spec**: [spec.md](spec.md)

## Summary

Correct the DistributedRepo packet data model so an original signed NDN Data
packet is authoritative under its complete encoded name. Logical object
manifests become indexes over exact packet names. The C++ embedded Repo and the
Python/SQLite network Repo will validate packet wire/name consistency, preserve
wire bytes, reject immutable-name conflicts, retrieve by exact name, activate
serving on the original prefix, and keep opaque-object APIs only as a separate
compatibility surface.

## Technical Context

**Language/Version**: C++17; Python 3.8+
**Dependencies**: ndn-cxx Data/Block/Name, NDNSF dynamic runtime, SQLite3,
Python sqlite3, MiniNDN/NFD
**Storage**: SQLite authority with exact packet table and manifest-reference
table; bounded exact-name hot cache
**Testing**: C++ packet-store test, Python SQLite/network tests, MiniNDN restart
and exact-Interest test
**Constraints**: Original Data wire is immutable; no re-signing, re-segmentation,
or Repo-derived packet names; no proposal-slide changes

## Constitution Check

- Dynamic NDNSF Request/Response remains the control path for insert, status,
  manifest discovery, and producer activation.
- Existing authorization, token, NAC-ABE, and trust behavior is unchanged.
- CodeGraph was used to trace `RepoClient`, `RepoNode::insertWirePackets`,
  `RepoNodeApp`, SQLite persistence, and `NativeWireDataProducer` before design.
- Spec Kit 074 owns the protocol/data-model correction and acceptance evidence.
- GSD Phase 6 records resumable implementation and verification.
- MiniNDN is the final network acceptance path.

## Architecture

### Canonical Data Model

```text
RepoObjectManifest(objectName, metadata)
  1 -> N ObjectPacketReference(objectName, ordinal, dataName)
  N -> 1 ExactDataPacket(dataName, wire, wireSha256, wireSize)
```

`ExactDataPacket.dataName` is parsed from `wire` by ndn-cxx. It is never
generated from `objectName`. The wire is stored once, while any number of
manifests may reference it.

### C++ Path

The existing store backend remains the authoritative opaque key/value engine.
Exact packets are stored with their complete Data name as the object key and
the original wire as payload. `RepoNode::insertWirePackets` decodes each wire,
validates names and packet-set shape, writes exact-name records, then stores a
parent manifest containing the ordered names. Explicit `putDataPacket` and
`getDataPacket` APIs prevent callers from accidentally using logical aliases.

The legacy `putSegmented/getSegmented` opaque-byte helper remains source
compatible but is documented as a non-NDN object chunking API. New packet APIs
never call it.

### Python/SQLite Path

Migrate from object-owned duplicated rows to:

```sql
data_packets(data_name PRIMARY KEY, wire, wire_sha256, wire_size, ...)
object_packet_refs(object_name, ordinal, segment_no, data_name,
                   PRIMARY KEY(object_name, ordinal))
```

Existing `data_segments` rows are migrated transactionally. Reads and writes
use the new tables. Manifest overwrite calculates old/new references in one
transaction. Unreferenced packets are reclaimed only after reference updates
commit.

### Serving

`FETCH_PREPARE` for a packet manifest loads exact wires and derives the common
original prefix from decoded packet names. `StoredDataProducer` is strengthened
to index packets by full name and only answers an Interest when the selected
Data name exactly equals the Interest name (or satisfies an explicitly allowed
prefix discovery Interest). It never selects solely by segment number across
different versions.

An exact `FETCH_PACKET_PREPARE(dataName)` control operation activates one stored
packet by its full name. Its response echoes the exact `dataName`, wire digest,
and original serving prefix. This supports callers that already know the
segment name without requiring a logical object lookup.

### Cache

The hot cache gains exact packet entries keyed by `dataName`. Packet-set cache
entries may remain as a serving optimization, but correctness and lookup do not
depend on object-keyed packet sets.

## Validation Sequence

1. Build C++ DistributedRepo and Python extension.
2. Run C++ exact packet tests including invalid wire and immutable conflicts.
3. Run Python migration, exact lookup, shared reference, restart, and producer
   name tests.
4. Run existing Repo/tiered-cache regressions.
5. Run MiniNDN: store custom `/data/.../v=.../seg=N` packets, restart Repo,
   activate/fetch exact segments, compare names and wire hashes, and confirm no
   Repo-derived alias exists.
6. Converge Spec tasks and verify GSD health.

## Acceptance Result

Completed on 2026-07-10. The final MiniNDN run stored four signed packets,
restarted Repo A, and passed exact-name, complete-wire identity, manifest index,
payload reassembly, restart persistence, and no-alias checks. Evidence is in
`results/distributed_repo_exact_packets_minindn/exact-packet-summary.json`.

## Project Structure

```text
specs/074-repo-exact-data-packets/
NDNSF-DistributedRepo/{include,src,examples,README*}
pythonWrapper/{src/ndnsf/_ndnsf.cpp,ndnsf/service.py}
NDNSF-DistributedInference/ndnsf_distributed_inference/repo.py
tests/python/test_ndnsf_repo_exact_packets.py
Experiments/NDNSF_DistributedRepo_ExactPackets_Minindn.py
```
