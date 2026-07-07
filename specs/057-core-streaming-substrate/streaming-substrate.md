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

The 5% lossy MiniNDN smoke uses `Experiments/Topology/UAV(loss=5%)`, where the
nodes are `drone1` and `gs1`:

```bash
python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
  --quick-smoke --no-cli --drone-headless --camera-mode file \
  --topology-file 'Experiments/Topology/UAV(loss=5%)' \
  --controller-node gs1 --gs-node gs1 --drone-node drone1

sudo -E timeout 220s xvfb-run -a \
  python3 Experiments/NDNSF_UAV_GUI_Minindn.py \
    --no-cli --no-xhost --drone-headless --camera-mode file \
    --topology-file 'Experiments/Topology/UAV(loss=5%)' \
    --controller-node gs1 --gs-node gs1 --drone-node drone1 \
    --auto-video-test --auto-stop-seconds 10 --auto-start-delay-ms 1000 \
    --video-bitrate-kbps 1200 --video-width 320 \
    --output-dir results/uav_core_stream_loss5_smoke_<timestamp>
```

The July 7, 2026 5% loss run under
`results/uav_core_stream_loss5_smoke_20260707_172517` also passed with
`NDNSF_UAV_GUI_MININDN_SMOKE_OK`. The drone published 42 stream packets across
12 FEC groups, the ground station logged `GS_DECODED_FRAMES count=30` and
reached 194 decoded frames before stop, and the ground station exited with
`GS_GUI_EXIT rc=0`.

## Distributed Inference Tensor Stream Smoke

The substrate is not video-specific. `Experiments/NDNSF_DI_LlmPipeline_Smoke.py`
uses core `StreamChunk` encoding for its fake LLM hidden-state dependency
store. Each planned dependency is published as one or more stream chunks with:

```text
contentType = application/x-ndnsf-di-tensor-bundle
metadata    = producer, consumer, keyScope, tensors, objectName
payload     = hidden-state bundle bytes
```

The consumer decodes each chunk, validates the stream id, sequence number,
content type, segment count, and object metadata, then reassembles the original
hidden-state bundle before executing the next fake pipeline stage.

Validation:

```bash
python3 Experiments/NDNSF_DI_LlmPipeline_Smoke.py \
  --out-dir /tmp/ndnsf-di-llm-pipeline-stream-smoke-default

python3 Experiments/NDNSF_DI_LlmPipeline_Smoke.py \
  --stream-chunk-bytes 17 \
  --out-dir /tmp/ndnsf-di-llm-pipeline-stream-smoke-small
```

Both runs passed on July 7, 2026. The default 64-byte chunk setting published
5 stream chunks for 2 dependencies; the forced 17-byte setting published 19
stream chunks. Both produced the same final output size and preserved the LLM
pipeline execution order.

## C++ Distributed Inference Large-Data Path

The real C++ DI collaboration path can also carry dependency tensors as core
`StreamChunk` payloads. `NdnsfCollaborationDependencyIo` keeps the old raw
payload behavior by default. When enabled, it wraps each complete tensor bundle
before `publishLargeNamed(...)` and unwraps it after `fetchLarge(...)`:

```text
contentType = application/x-ndnsf-di-tensor-bundle
streamId    = plannedDataName
seq         = 0
segment     = 0 of 1
metadata    = sessionId, scope, producerRole, consumerRole, plannedDataName,
              bundleName, tensors
payload     = tensor bundle bytes
```

The mode is enabled through either:

```cpp
NativeProviderHandlerConfig config;
config.streamChunkDependencies = true;
```

or the runtime environment:

```bash
NDNSF_DI_STREAM_CHUNK_DEPENDENCIES=1
```

Validation added on July 7, 2026:

```bash
./waf build --targets=unit-tests
./build/unit-tests --run_test=NdnsfCollaborationDependencyIoWrapsTensorBundleAsStreamChunk
./build/unit-tests --run_test=NdnsfCollaborationDependencyIoRejectsInvalidStreamChunkPayload
./build/unit-tests --run_test=Stream
PYTHONPATH=pythonWrapper python3 tests/python/test_ndnsf_core_streaming.py
./build/unit-tests
```

The full `unit-tests` run passed with 185 test cases. This proves the new C++
StreamChunk tensor-bundle envelope is available without changing default raw
DI dependency behavior.

## Next Design: Real DI StreamChunk Validation Campaign

The next batch moves the StreamChunk DI work from local/unit proof to the real
NDNSF-DI network path. The design principle is conservative:

```text
default mode       = raw tensor bundle payload
experimental mode  = StreamChunk tensor bundle envelope
decision point     = enable by default only after real-network evidence
```

The campaign should answer four questions:

1. Does `NDNSF_DI_STREAM_CHUNK_DEPENDENCIES=1` work end-to-end in the same
   MiniNDN Qwen/NativeTracer path used by NDNSF-DI experiments?
2. Do the final outputs and dependency execution order match the raw baseline?
3. How much wire-size, latency, and failure-rate overhead does the StreamChunk
   envelope add?
4. Does the feature help future stream/counter/debug reuse enough to justify
   turning it on by default?

### Runtime Path

The full-network validation must use the real C++ provider path:

```text
NativeProviderHandler
  -> NdnsfCollaborationDependencyIo
  -> CollaborationContext.publishLargeNamed(...)
  -> NDNSF large-data path
  -> CollaborationContext.fetchLarge(...)
  -> NdnsfCollaborationDependencyIo
```

The test is not considered sufficient if it only uses an in-memory dependency
store or the Python fake LLM pipeline. Those remain useful smoke tests, but the
next gate is the real `publishLargeNamed/fetchLarge` path.

### Mode Matrix

Run the same workload in both modes:

```text
raw mode:
  NDNSF_DI_STREAM_CHUNK_DEPENDENCIES=0

StreamChunk mode:
  NDNSF_DI_STREAM_CHUNK_DEPENDENCIES=1
```

At minimum, use the current smallest Qwen/NativeTracer artifacts and the
existing MiniNDN full-network harness. If a full Qwen model run is too slow for
iteration, use a short NativeTracer correctness smoke first, then run the
smallest Qwen configuration as the acceptance test.

### Required Evidence

Each run should emit a compact evidence bundle:

```text
config.json
run.log
request_lifecycle.csv or equivalent sampled lifecycle trace
dependency_lifecycle.csv
streamchunk_counters.json
summary.json
```

The summary should include:

```text
mode
requests offered
requests completed
failure rate
p50/p95/p99 latency
dependency fetch p50/p95
published dependency bytes
fetched dependency bytes
StreamChunk envelope bytes
overhead ratio
timeout count
decode rejection count
final output hash or exact output text
```

### Diagnostics

The C++ path should produce grep-friendly counters when
`NDNSF_DI_RUNTIME_TIMING=1` or a new stream-specific diagnostics flag is set:

```text
NDNSF_DI_STREAM_DEPENDENCY
  session=<id>
  scope=<scope>
  mode=raw|streamchunk
  direction=publish|fetch
  payload_bytes=<n>
  wire_bytes=<n>
  envelope_bytes=<n>
  planned_name=<name>
  status=ok|decode-error|timeout
```

This is intentionally a low-friction log interface first. A richer metrics API
can come later if the evidence shows the mode is worth promoting.

### Benchmark Policy

The first comparison should be small but reproducible:

```text
loss: 0%
requests: short smoke plus small campaign
modes: raw and StreamChunk
repetitions: at least 3 for smoke, 10 for final small campaign if runtime allows
```

Only after the 0% run is stable should we add 1-5% loss. The purpose is not to
claim StreamChunk improves loss recovery; the purpose is to confirm the
envelope does not make NDNSF-DI less robust under normal experimental loss.

### Enable-By-Default Rule

Keep StreamChunk dependency mode off by default until all of these are true:

- raw and StreamChunk modes produce matching outputs;
- StreamChunk mode has no new decode failures or hangs;
- overhead is measured and acceptable for tensor payload sizes used by
  NDNSF-DI;
- at least one real MiniNDN full-network run passes in StreamChunk mode;
- documentation explains when to use raw mode versus StreamChunk mode.

If these gates pass, the next spec can decide whether to flip the default or
keep it as an explicit debug/interoperability mode.

## DI StreamChunk Validation Campaign Result

Completed on July 7, 2026. The canonical full-network entrypoint is:

```bash
PYTHONPATH=NDNSF-DistributedInference:pythonWrapper:. \
python3 Experiments/NDNSF_DI_NativeTracer_Minindn.py \
  --out results/streamchunk_mode_raw_smoke_20260707c \
  --full-network \
  --tracer-deterministic-runner \
  --dependency-payload-mode raw \
  --requests 1 \
  --concurrency 1 \
  --provider-check-timeout 60
```

The StreamChunk mode uses the same harness and workload, changing only:

```bash
--dependency-payload-mode streamchunk
```

The harness maps this to provider environment:

```text
raw:
  NDNSF_DI_STREAM_CHUNK_DEPENDENCIES=0
  NDNSF_DI_STREAM_DEPENDENCY_TRACE=1

streamchunk:
  NDNSF_DI_STREAM_CHUNK_DEPENDENCIES=1
  NDNSF_DI_STREAM_DEPENDENCY_TRACE=1
```

The accepted campaign wrapper is:

```bash
sudo -E timeout 1800s env PYTHONPATH=NDNSF-DistributedInference:pythonWrapper:. \
python3 Experiments/NDNSF_DI_StreamChunk_Mode_Campaign.py \
  --out results/streamchunk_mode_campaign_3rep_20260707 \
  --modes raw,streamchunk \
  --repeats 3 \
  --requests 1 \
  --concurrency 1 \
  -- \
  --tracer-deterministic-runner \
  --provider-check-timeout 60
```

Accepted 0% loss result:

```text
results/streamchunk_mode_campaign_3rep_20260707
```

Summary:

| Mode | Repeats | Success | Mean p50 ms | Decode errors | Dependency events | Output hash |
|---|---:|---:|---:|---:|---:|---|
| raw | 3 | 3/3 | 260.054 | 0 | 24 | matched |
| streamchunk | 3 | 3/3 | 268.555 | 0 | 24 | matched |

Per run, the NativeTracer plan produced 8 dependency events: 4 publish events
and 4 fetch events across the real NDNSF large-data path. Provider logs include
both existing `NDNSF_DI_DEPENDENCY_OUTPUT_TIMING` /
`NDNSF_DI_DEPENDENCY_INPUT_TIMING` entries and the new
`NDNSF_DI_STREAM_DEPENDENCY` entries. This confirms the validation used
`publishLargeNamed(...)` and `fetchLarge(...)`, not the earlier in-memory
dependency store.

For this tiny deterministic tensor payload, StreamChunk mode added a visible
wire envelope:

```text
raw payload bytes:        492
raw wire bytes:           492
stream payload bytes:     492
stream wire bytes:        4892
stream envelope bytes:    4400
stream overhead ratio:    8.943089
```

This overhead ratio is high because the smoke payloads are intentionally tiny
NativeTracer artifacts. The result is still useful: it proves correctness,
metadata preservation, decode rejection plumbing, and measurement support. It
does not prove that StreamChunk should be faster or smaller than raw tensor
payloads.

## Low-Loss Robustness Smoke

The harness now accepts an explicit topology path:

```bash
--topology-file Experiments/Topology/AI_Lab(loss=1%).conf
```

The low-loss topology keeps the same AI Lab nodes and adds `loss=1` to each
link. Accepted low-loss command:

```bash
sudo -E timeout 700s env PYTHONPATH=NDNSF-DistributedInference:pythonWrapper:. \
python3 Experiments/NDNSF_DI_StreamChunk_Mode_Campaign.py \
  --out results/streamchunk_mode_loss1_campaign_20260707 \
  --modes raw,streamchunk \
  --repeats 1 \
  --requests 1 \
  --concurrency 1 \
  -- \
  --tracer-deterministic-runner \
  --provider-check-timeout 60 \
  --topology-file 'Experiments/Topology/AI_Lab(loss=1%).conf'
```

Accepted 1% loss result:

```text
results/streamchunk_mode_loss1_campaign_20260707
```

Summary:

| Mode | Success | p50 ms | Timeout count | Decode errors | Output hash |
|---|---:|---:|---:|---:|---|
| raw | 1/1 | 721.773 | 0 | 0 | matched |
| streamchunk | 1/1 | 1495.472 | 0 | 0 | matched |

The low-loss smoke shows no StreamChunk decode failures, hangs, or new timeout
pattern in this one-request validation. It should not be read as a performance
improvement. StreamChunk mode was slower in this lossy smoke, consistent with
the extra envelope bytes and the already-known bounded-time behavior of
SVS-based service invocation under loss.

## Raw Versus StreamChunk Dependency Mode

Keep raw dependency mode as the default.

Use raw mode when:

- the dependency payload is already an application-specific tensor bundle;
- throughput and latency are more important than reusable stream metadata;
- the experiment does not need stream-level diagnostics or cross-application
  stream/FEC/reorder semantics.

Use StreamChunk dependency mode when:

- the experiment needs app-neutral stream metadata on DI dependencies;
- the same diagnostics/counter tools should work across video streams and DI
  tensor streams;
- the payload must carry explicit session, scope, producer, consumer,
  planned-data-name, content-type, and segment metadata;
- the goal is interoperability or debugging, not minimum wire overhead.

The default remains opt-in because the validation shows correctness, but also
measures non-trivial envelope overhead on tiny NativeTracer payloads. The
existing opt-in path is:

```bash
--dependency-payload-mode streamchunk
```

or, for direct provider configuration:

```bash
NDNSF_DI_STREAM_CHUNK_DEPENDENCIES=1
```

No default flip was made, so no new opt-out flag is required.

## StreamChunk Dependency Troubleshooting

Useful grep patterns:

```bash
rg "NDNSF_DI_STREAM_DEPENDENCY|NDNSF_DI_DEPENDENCY_INPUT_TIMING|NDNSF_DI_DEPENDENCY_OUTPUT_TIMING" \
  results/<run>/logs
```

Common failures:

- `content type mismatch`: the payload decoded as `StreamChunk`, but its
  `contentType` is not `application/x-ndnsf-di-tensor-bundle`.
- `segment mismatch`: DI dependency mode currently expects one complete tensor
  bundle per StreamChunk envelope, with `segmentIndex=0` and `segmentCount=1`.
- `scope/session mismatch`: metadata in the received StreamChunk does not match
  the planned dependency edge or active request session.
- `timeout`: check whether the provider published
  `NDNSF_DI_DEPENDENCY_OUTPUT_TIMING` for the planned name before the consumer
  logged input timing. If publication exists but fetch times out, inspect
  topology loss, NFD routes, and SVS bounded-time delivery behavior.
- `decode-error`: inspect the matching `NDNSF_DI_STREAM_DEPENDENCY` entry for
  mode, planned name, and wire bytes. The harness counts these in
  `streamchunk_counters.json` and `summary.json`.
