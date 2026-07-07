# Tasks: Core Streaming Substrate

**Input**: Design documents from `specs/057-core-streaming-substrate/`

**Prerequisites**: `spec.md`, `plan.md`

**Tests**: Required because this is a reusable core API.

## Phase 1: Setup

- [x] T001 Create `specs/057-core-streaming-substrate/` with spec, plan, and tasks.
- [x] T002 Confirm CodeGraph, GSD, Spec Kit, and DeepSeek decision gates.

## Phase 2: Foundational Core API

- [x] T003 [P] Create `ndn-service-framework/Stream.hpp` and `Stream.cpp` with app-neutral C++ stream entities.
- [x] T004 [P] Implement C++ TLV encoding/decoding for stream info, chunks, and FEC metadata.
- [x] T005 [P] Add `pythonWrapper/ndnsf/streaming.py` as a thin orchestration/test mirror.
- [x] T006 Export streaming entities from `pythonWrapper/ndnsf/__init__.py`.

## Phase 3: Producer And Consumer Helpers

- [x] T007 Implement bounded `StreamProducerBuffer`.
- [x] T008 Implement `StreamConsumerReorderBuffer` with current-session, duplicate, and in-order emission behavior.
- [x] T009 Implement codec-neutral `StreamFecInfo` metadata helpers.

## Phase 4: Adaptive Fetch State

- [x] T010 Implement `StreamAdaptiveFetcherState` and `StreamFetchDecision`.
- [x] T011 Add policy helper for window, lookahead, interest lifetime, and missing timeout decisions.

## Phase 5: Tests

- [x] T012 [P] Add `tests/unit-tests/stream.t.cpp` and `tests/python/test_ndnsf_core_streaming.py` for metadata round trips.
- [x] T013 [P] Add C++/Python tests for producer buffer eviction and lookup.
- [x] T014 [P] Add C++/Python tests for reorder, duplicate, stale-session, and missing-gap behavior.
- [x] T015 [P] Add C++/Python tests for adaptive fetch decisions.

## Phase 6: Documentation

- [x] T016 Add `specs/057-core-streaming-substrate/streaming-substrate.md` with NDNSF/UAV boundary and migration mapping.
- [x] T017 Update `pythonWrapper/README.md` and `pythonWrapper/README_ch.md` with a short API entry.
- [x] T018 Update `NDNSF-UAV-APP/README.md` and `NDNSF-UAV-APP/README_ch.md` to reference the core substrate boundary.

## Phase 7: Verification

- [x] T019 Run C++ core streaming tests.
- [x] T020 Run Python streaming and existing core coordination tests.
- [x] T021 Review DeepSeek checklist against final implementation and record accepted/rejected suggestions.

## Phase 8: UAV Compatibility Mapping

- [x] T022 Add `VideoPacket` to core `StreamChunk` conversion helpers in `NDNSF-UAV-APP/shared/UavProtocol.*`.
- [x] T023 Add UAV protocol test proving conversion preserves stream/session/frame/FEC fields.
- [x] T024 Add UAV protocol test proving `encodeVideoPacket(...)` output is unchanged after `VideoPacket -> StreamChunk -> VideoPacket`.
- [x] T025 Run `UavProtocolState`, `Stream`, Python streaming tests, and full `./waf build`.

## Phase 9: UAV Producer Migration

- [x] T026 Update drone `publishCurrentFrame()` to construct core `StreamChunk` values for data shards.
- [x] T027 Update drone `publishCurrentFrame()` to construct core `StreamChunk` values for parity shards.
- [x] T028 Keep final publication on the existing `streamChunkToVideoPacket(...) -> encodeVideoPacket(...)` compatibility path.
- [x] T029 Run `UavProtocolState`, `Stream`, Python streaming tests, and full `./waf build` after the producer migration.

## Phase 10: UAV Consumer Handoff Migration

- [x] T030 Rename the ground-station decoder queue item to `DecoderStreamChunk` so it no longer collides with core `StreamChunk`.
- [x] T031 Add `insertStreamChunkForDecode(...)` as the single core `StreamChunk` handoff into the existing decoder queue.
- [x] T032 Convert non-FEC, FEC data-shard, and recovered FEC output paths to enter the decoder through core `StreamChunk`.
- [x] T033 Keep existing FEC recovery and decoder queue ordering intact; do not place core duplicate/reorder filtering before FEC.
- [x] T034 Run `UavProtocolState`, `Stream`, Python streaming tests, and full `./waf build` after the consumer handoff migration.

## Phase 11: Receiver-Side FEC Handoff Evidence

- [x] T035 Add a pure C++ receiver-side FEC handoff test using `VideoPacket -> StreamChunk` mapping.
- [x] T036 Verify data and parity `StreamChunk` values preserve enough metadata and payload to recover a missing data shard.
- [x] T037 Run `UavProtocolState`, `Stream`, Python streaming tests, and full `./waf build` after adding the handoff evidence.

## Phase 12: MiniNDN UAV Live-Stream Smoke

- [x] T038 Run the UAV MiniNDN quick smoke for binary/config/topology readiness.
- [x] T039 Run headless-drone auto video smoke through `Experiments/NDNSF_UAV_GUI_Minindn.py`.
- [x] T040 Verify the live stream starts, produces drone-owned video Data, decodes frames at the ground station, and stops cleanly.

## Phase 13: Durable Documentation

- [x] T041 Move the streaming substrate boundary and live-smoke recipe into tracked feature documentation.
- [x] T042 Keep ignored `docs/` output out of commits while preserving the evidence in `specs/057-core-streaming-substrate/streaming-substrate.md`.

## Phase 14: Lossy MiniNDN Stream Smoke

- [x] T043 Run the UAV MiniNDN quick smoke on `Experiments/Topology/UAV(loss=5%)`.
- [x] T044 Run headless-drone auto video smoke on the 5% loss topology.
- [x] T045 Record decoded-frame, packet, FEC-group, and clean-stop evidence for the 5% loss run.
