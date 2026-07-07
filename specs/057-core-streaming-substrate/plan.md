# Implementation Plan: Core Streaming Substrate

**Branch**: `057-core-streaming-substrate` | **Date**: 2026-07-07 | **Spec**: [spec.md](spec.md)

## Summary

Add a reusable, app-neutral NDNSF streaming substrate in the C++ core library,
with a thin Python mirror for orchestration and tests. The core defines stream
session/chunk/FEC/metrics contracts, TLV chunk metadata encoding, bounded
producer buffering, consumer reorder/stale/duplicate handling, and adaptive
fetch state. NDNSF-UAV-APP remains behavior-compatible: its existing C++ video
stream is treated as the reference application mapping, not as code to move
wholesale into core.

## Technical Context

**Language/Version**: C++17 for the primary core substrate; Python 3.8+ for a thin wrapper/test mirror.

**Primary Dependencies**: ndn-cxx Block/TLV helpers and Python standard library.

**Storage**: In-memory helper buffers only.

**Testing**: Boost unit tests under `tests/unit-tests` and Python `unittest` tests under `tests/python`.

**Target Platform**: Linux source-tree development and MiniNDN experiments.

**Project Type**: Core library feature in `ndn-service-framework`, with Python wrapper exports.

**Performance Goals**: Deterministic chunk lookup by sequence and bounded memory via explicit max chunk counts.

**Constraints**: Do not move UAV H264/camera/decoder logic into NDNSF core. Do not change NDNSF security flow or existing UAV packet names in this iteration.

**Scale/Scope**: Generic API sufficient for UAV video mapping, telemetry/log streams, DI intermediate streams, and future multi-part service responses.

## Boundary Decision

NDNSF C++ core owns:

- stream identity: stream id, session epoch, stream prefix, next sequence;
- generic chunk metadata: sequence, capture time, deadline, payload size, content type, key marker, opaque application metadata;
- optional codec-neutral FEC metadata;
- producer buffer and consumer reorder helper behavior;
- generic adaptive fetch decision state;
- TLV encoding for stream info and stream chunks.

Applications own:

- camera capture, ffmpeg, H264, ROI, YOLO, and video decoder queues;
- model/tensor semantics and DI role scheduling;
- actual FEC codec implementation and repair policy;
- bitrate policy that changes application encoder settings.

## Project Structure

```text
ndn-service-framework/
├── Stream.hpp            # primary C++ stream substrate
└── Stream.cpp

pythonWrapper/ndnsf/
├── streaming.py          # thin Python mirror for orchestration/tests
└── __init__.py           # public exports

tests/unit-tests/
└── stream.t.cpp

tests/python/
└── test_ndnsf_core_streaming.py

specs/057-core-streaming-substrate/
├── spec.md
├── plan.md
├── streaming-substrate.md
└── tasks.md
```

## Constitution Check

- **Canonical Dynamic Runtime**: PASS. The feature adds a reusable helper layer and does not reintroduce generated service/stub APIs.
- **Security Is Part Of The Data Path**: PASS. Stream start/control can still use existing NDNSF service auth, while stream chunk Data remains signed or encrypted by the application/runtime path.
- **CodeGraph First, Source Verified**: PASS. CodeGraph was used before planning and implementation.
- **Spec-Driven Changes For Durable Work**: PASS. This feature has an independent spec, plan, and task list.
- **Verify With The Right Scope**: PASS. C++ and Python tests cover the reusable contract; UAV behavior migration is intentionally out of scope for this iteration.

## Validation

```bash
./waf --run unit-tests --run-args='--run_test=Stream'
PYTHONPATH=pythonWrapper python3 tests/python/test_ndnsf_core_streaming.py
PYTHONPATH=pythonWrapper python3 tests/python/test_ndnsf_core_coordination.py
```

The live MiniNDN validation recipe and July 7, 2026 smoke evidence are recorded
in [streaming-substrate.md](streaming-substrate.md).
