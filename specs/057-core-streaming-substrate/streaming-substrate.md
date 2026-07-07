# NDNSF Streaming Substrate

NDNSF now has an app-neutral streaming substrate in the C++ core library:

```text
ndn-service-framework/Stream.hpp
ndn-service-framework/Stream.cpp
```

The goal is to keep reusable stream/session/chunk behavior in NDNSF while
leaving application-specific media, inference, or workload logic in the
application.

## Boundary

NDNSF core owns:

- stream identity: `streamId`, `sessionEpoch`, `streamPrefix`, and `nextSeq`;
- generic chunk metadata: sequence number, content type, capture time, arrival
  time, deadline, frame/segment hints, key-chunk marker, and opaque metadata;
- optional codec-neutral FEC metadata through `StreamFecInfo`;
- bounded producer buffering through `StreamProducerBuffer`;
- consumer stale-session, duplicate, gap, and in-order handling through
  `StreamConsumerReorderBuffer`;
- generic adaptive fetch state through `StreamAdaptiveFetcherState`;
- TLV wire encoding for `StreamInfo`, `StreamChunk`, and `StreamFecInfo`.

Applications own:

- camera capture, ffmpeg, H264/H265, ROI, YOLO, and decoder queues;
- tensor, model, or workload-specific payload semantics;
- the actual FEC repair algorithm;
- bitrate or encoder control policy;
- GUI presentation and application-specific logs.

This keeps the core reusable for UAV video, telemetry streams, log streams,
distributed-inference intermediate streams, and future multi-part service
responses.

## UAV Mapping

The current `NDNSF-UAV-APP` live video path maps naturally to the core stream
contract:

```text
NDNSF service invocation
  /<drone>/UAV/Camera/Video start/stop
  returns stream_id, stream_session_epoch, stream_prefix, bitrate/fps/FEC hints

Named Data stream
  /<drone>/video/<stream-start-ms>/<packetSeq>
  each Data carries one video chunk plus metadata
```

The bridge helpers are:

```cpp
StreamChunk videoPacketToStreamChunk(const VideoPacket& packet);
VideoPacket streamChunkToVideoPacket(const StreamChunk& chunk);
```

They preserve the existing UAV video wire format. `encodeVideoPacket` and
`decodeVideoPacket` still produce and consume the same compact UAV header,
while new code can pass equivalent stream semantics through core `StreamChunk`.

The drone publisher now builds core `StreamChunk` values for data and parity
shards, then converts them back to the existing `VideoPacket` wire encoder.
The ground station keeps existing FEC recovery and decoder queue semantics, but
every chunk that enters the decoder is first represented as a core
`StreamChunk`.

The core substrate does not replace UAV's H264 encoder, XOR recovery, decoder,
or bitrate policy.

## Validation

Focused local validation:

```bash
./waf build --targets=unit-tests
./build/unit-tests --run_test=Stream
./build/unit-tests --run_test=UavProtocolState
PYTHONPATH=pythonWrapper python3 tests/python/test_ndnsf_core_streaming.py
PYTHONPATH=pythonWrapper python3 tests/python/test_ndnsf_core_coordination.py
./waf build
```

MiniNDN live-stream smoke:

```bash
python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --quick-smoke --no-cli --drone-headless --camera-mode file

sudo -E timeout 160s xvfb-run -a \
  python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
    --no-cli --no-xhost --drone-headless --camera-mode file \
    --auto-video-test --auto-stop-seconds 6 --auto-start-delay-ms 1000 \
    --video-bitrate-kbps 1200 --video-width 320 \
    --output-dir results/uav_core_stream_smoke_<timestamp>
```

The July 7, 2026 smoke run under
`results/uav_core_stream_smoke_20260707_171937` passed with
`NDNSF_UAV_GUI_MININDN_SMOKE_OK`. The drone published 29 live stream packets,
the ground station logged `GS_DECODED_FRAMES count=30` and reached 88 decoded
frames before stop, and the drone logged both `video streaming` and
`video stopped`.
