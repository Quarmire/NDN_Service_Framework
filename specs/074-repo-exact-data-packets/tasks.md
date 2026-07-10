# Tasks: Exact NDN Data Packet Repository

## Phase 1 - Contract and fixtures

- [x] T001 Add exact-packet test fixtures with custom base, version, segments,
  declared names, wire hashes, duplicate references, and conflict wires.
- [x] T002 Add failing C++ tests proving packet APIs currently create aliases or
  cannot retrieve by exact Data name.
- [x] T003 Add failing Python tests for exact SQLite lookup, restart, shared
  references, conflict rejection, and original-prefix preparation.

## Phase 2 - C++ exact packet authority

- [x] T004 Add an `ExactDataPacket` C++ type and ndn-cxx wire decoder that derives
  the complete name, segment number, and wire digest.
- [x] T005 Add explicit backend/core/node/client APIs to store, test, retrieve,
  and delete packets by exact Data name.
- [x] T006 Change `insertWirePackets` to store exact names and a manifest index;
  remove `/ndn-data/N` generation from the packet path.
- [x] T007 Make idempotent reinsertion succeed and conflicting same-name wire fail
  with a stable operation status.
- [x] T008 Preserve the old opaque `putSegmented/getSegmented` helper as clearly
  labeled compatibility behavior without calling it from packet APIs.

## Phase 3 - Python SQLite authority and cache

- [x] T009 Add `data_packets` and `object_packet_refs` tables and transactional
  migration from existing `data_segments` rows.
- [x] T010 Decode and validate packet wire/name/hash before writes; implement
  idempotency and immutable-name conflict rejection.
- [x] T011 Replace object-owned packet persistence/load/delete with exact packet
  rows, ordered references, and safe orphan reclamation.
- [x] T012 Add exact-name packet cache lookup and counters while retaining the
  shared bounded cache budget.

## Phase 4 - Original-name network serving

- [x] T013 Change native `StoredDataProducer` to index by full Data name and
  reject segment-only cross-version selection.
- [x] T014 Derive packet-set serving prefixes/versioned names from stored packet
  names; never use `RepoNodeApp.data_name()` for stored packet wires.
- [x] T015 Add `FETCH_PACKET_PREPARE(dataName)` and client helper for exact known
  segment retrieval.
- [x] T016 Return authoritative `packetNames` and versioned Data name from
  manifest `FETCH_PREPARE` and validate them client-side.
- [x] T017 Update catalog repair/replication to preserve exact packet names and
  wire bytes end-to-end.

## Phase 5 - Regression and documentation

- [x] T018 Pass focused C++ exact packet tests and existing Repo smoke/cache tests.
- [x] T019 Pass Python exact packet, tiered cache, Repo envelope, and packet
  storage regressions.
- [x] T020 Update English/Chinese Repo documentation, examples, and config help
  to state exact packet versus opaque object semantics.

## Phase 6 - MiniNDN acceptance and convergence

- [x] T021 Add a MiniNDN scenario that stores signed packets under
  `/data/.../v=.../seg=N`, restarts the Repo, and fetches exact names.
- [x] T022 Verify packet names, complete wire hashes, manifest ordering,
  same-name conflict handling, cache read-through, and absence of aliases.
- [x] T023 Record a machine-readable acceptance summary and exact reproduction
  command.
- [x] T024 Run Spec Kit convergence, GSD health validation, and document any
  residual compatibility risk.
