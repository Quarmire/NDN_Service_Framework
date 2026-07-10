# Implementation Plan: Repo Packet Consumer Contract

**Spec**: `specs/075-repo-packet-consumer-contract/spec.md`
**Date**: 2026-07-10

## Summary

Add symmetric packet-set retrieval APIs on top of Spec 074 exact-name storage.
The C++ and Python clients will consume `packetNames` in order, validate exact
name/wire identity, and fail atomically. Opaque-object APIs remain unchanged
and the existing payload view remains compatible. The Repo example and MiniNDN
acceptance path will use the new public operation.

## Technical Context

- C++17, ndn-cxx `ndn::Data`, existing `RepoClient` and `RepoObjectManifest`.
- Python 3, native `DataPacket`, `NetworkDistributedRepoClient`, and
  `DistributedRepo` facade.
- SQLite exact packet authority and native producer from Spec 074.
- Existing waf build and Python unittest/pytest suites.
- MiniNDN remains the network acceptance environment.

## Constitution Check

- **Canonical runtime**: No generated API or legacy service path is added.
- **Security**: Stored Data wires and signatures remain unchanged.
- **CodeGraph first**: Callers and impact surface were audited before design.
- **Spec-driven**: This plan, contracts, tasks, and acceptance guide are durable.
- **Verification**: Focused C++/Python tests plus MiniNDN exact-packet acceptance.

No constitution violations are required.

## Design

### C++ API

Add `RepoClient::getDataPackets(node, manifest)`. It validates that
`packetNames` is non-empty, unique, and consistent with `segmentCount`, then
calls exact `getDataPacket` in manifest order. Each wire is decoded as
`ndn::Data`; its complete name must equal the manifest entry.

### Python API

Add `NetworkDistributedRepoClient.fetch_signed_packets(manifest)` and
`DistributedRepo.get_signed_packets(...)`. The operation chooses a declared
replica, retrieves each exact name through `fetch_packet`, validates count,
uniqueness, exact name, and wire digest consistency, and returns only after the
whole set succeeds.

### Two Read Views

`fetch_signed_packets(...)` returns the original packet wires. Existing
`fetch_object(..., manifest)` remains a payload view and may use SegmentFetcher
over the original stored packet names. The payload view does not authorize the
Repo to rename, re-sign, or persist a second representation.

### Application Classification

- Generic Repo signed packet example: migrate to batch packet retrieval.
- DI model/runtime artifacts: remain opaque objects.
- UAV encrypted recording chunks: remain opaque objects.
- Future UAV/DI producers that already create signed Data: use packet APIs.

## Project Structure

```text
NDNSF-DistributedRepo/include/.../RepoClient.hpp
NDNSF-DistributedRepo/src/RepoClient.cpp
NDNSF-DistributedRepo/examples/DistributedRepoExactPacketTest.cpp
NDNSF-DistributedInference/ndnsf_distributed_inference/repo.py
examples/python/NDNSF-DistributedRepo/generic_object_store/client.py
tests/python/test_ndnsf_repo_exact_packets.py
NDNSF-DistributedRepo/README.md
NDNSF-DistributedRepo/README_ch.md
Experiments/NDNSF_DistributedRepo_Generic_Minindn.py
```

## Verification Strategy

1. C++ exact packet test covers order, complete-name decoding, duplicate index,
   count mismatch, and missing packet rejection.
2. Python tests cover batch success and all atomic failure modes while the
   existing payload view remains compatible.
3. Existing C++ smoke/cache and Python tiered-cache/discovery tests remain green.
4. MiniNDN exact-packet smoke uses the batch API and retains exact names, wire
   identity, restart persistence, and no-alias evidence.

## Post-Design Constitution Check

The final design narrows APIs without changing NDNSF security or application
ownership boundaries. The exact packet contract is additive; legacy opaque
chunking remains isolated and is not used by packet consumers.
