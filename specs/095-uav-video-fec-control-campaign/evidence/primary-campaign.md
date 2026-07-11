# Primary MiniNDN Campaign Evidence

## Reproduction

Corrected immutable result directory:

```text
results/spec095-uav-video-fec-control-corrected-primary
```

Command used once, with no automatic retry:

```bash
python3 Experiments/NDNSF_UAV_Stream_Parity_Campaign.py \
  --out results/spec095-uav-video-fec-control-corrected-primary \
  --runs 3 \
  --loss-percentages 0,5 \
  --fec-parity-shards 0,1 \
  --auto-stop-seconds 60
```

After adding the usable-frame gate, the same logs were re-parsed without
executing MiniNDN again:

```bash
python3 Experiments/NDNSF_UAV_Stream_Parity_Campaign.py \
  --out results/spec095-uav-video-fec-control-corrected-primary \
  --runs 3 \
  --loss-percentages 0,5 \
  --fec-parity-shards 0,1 \
  --auto-stop-seconds 60 \
  --reparse-existing
```

The reparse exits nonzero by design because 3 of 12 runs do not satisfy the
combined acceptance gate. It does not replace or rerun them.

## Results

| One-way loss | XOR parity | Combined/video/control | Mean decoded frames | Mean recovered chunks | Mean RTT p95 | Mean timeouts |
|---:|---:|---:|---:|---:|---:|---:|
| 0% | 0 | 3/3 | 1710 | 0.00 | 81.67 ms | 34.67 |
| 0% | 1 | 3/3 | 1710 | 0.33 | 107.43 ms | 45.33 |
| 5% | 0 | 2/3 | 1070 | 0.00 | 97.80 ms | 26.00 |
| 5% | 1 | 1/3 | 670 | 8.33 | 105.33 ms | 13.67 |

Every run lasted about 61-62 seconds. All runs stayed below the fixed buffering
limits; the maximum observed state was 8 pending chunks and 39,600 pending
bytes. No stale-session or stale-stream acceptance was observed.

## Interpretation

At 0% loss, both treatments completed all repetitions. Parity did not improve
decoded delivery and coincided with higher mean RTT p95 and timeout count.

At 5% loss, XOR parity demonstrably recovered chunks, but this did not improve
usable video or concurrent Targeted control completion. FEC-off completed two
of three repetitions; FEC-on completed one of three. This is a negative result,
not evidence that parity caused the regression: three repetitions are too few
for a causal or statistical claim, and both video and control share the same
lossy NDNSF/NDN environment.

The evidence supports only these bounded conclusions:

1. The UAV request can select data-only or one-XOR-parity publication.
2. The parity receiver recovers some missing chunks under loss.
3. This simple parity setting did not improve end-to-end application success in
   the primary 5% MiniNDN campaign.
4. Application acceptance must include delivered video rate and control results,
   not process exit or FEC recovery counters alone.

## Limitations And Residual Risk

- MiniNDN models link loss; it is not a real wireless-radio campaign.
- The source is the file-camera H264 path, not a physical camera.
- MAVLink uses the existing automated/fake-flight-controller path rather than a
  real PX4 vehicle.
- Three repetitions permit descriptive comparison only.
- The campaign does not isolate whether the 5% failures originate in stream
  synchronization, Targeted control delivery, shared control-plane contention,
  or their interaction.
- The optional 15% boundary cells were not run because the primary 5% matrix
  already exposed the boundary and the spec does not require extra cells.
