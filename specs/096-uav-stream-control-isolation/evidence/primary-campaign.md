# Primary MiniNDN Isolation Campaign

## Reproduction

Result directory:

```text
results/spec096-uav-stream-control-isolation-primary
```

The campaign was executed once with no automatic retry or replacement:

```bash
python3 Experiments/NDNSF_UAV_Stream_Control_Isolation_Campaign.py \
  --out results/spec096-uav-stream-control-isolation-primary \
  --runs 3 \
  --loss-percent 5 \
  --auto-stop-seconds 60
```

The command returned 1 because seven runs failed their registered acceptance
gate. All 15 unique run directories and outcomes remain in JSON/CSV.

## Results

| Workload | Accepted | Video | Control | Mean decoded | Mean FEC recovered | Mean RTT p95 |
|---|---:|---:|---:|---:|---:|---:|
| control-only | 0/3 | n/a | 0/3 | n/a | n/a | n/a |
| video-only FEC0 | 2/3 | 2/3 | n/a | 1420 | 0 | 92.67 ms |
| video-only FEC1 | 3/3 | 3/3 | n/a | 1640 | 23 | 84.87 ms |
| combined FEC0 | 1/3 | 3/3 | 1/3 | 1600 | 0 | 92.33 ms |
| combined FEC1 | 2/3 | 2/3 | 2/3 | 1160 | 14 | 95.97 ms |

Across parity settings, video-only completed 5/6 video components and combined
also completed 5/6. Control-only completed 0/3 command sequences; combined
completed 3/6. Every structured video metric was valid. Maximum observed
buffer state was 9 chunks and 54,000 bytes, far below the fixed limits, and no
stale stream/session acceptance was observed.

Failure details:

- `control-only-run-01`: no Arm/Takeoff/Land acceptance; process aborted after
  `GS_GUI_EXIT rc=0` with `terminate called without an active exception`.
- `control-only-run-02` and `run-03`: no Arm/Takeoff/Land acceptance; launcher
  failed on missing Arm marker. Emergency-stop responses were observed.
- `video-only-fec0-run-03`: 1,350 frames, but only 51.27 seconds of the requested
  stream duration; launcher also missed the active-view marker.
- `combined-fec0-run-01` and `run-03`: video passed, Arm/Land passed, Takeoff
  was missing.
- `combined-fec1-run-03`: 120 frames and no three control markers; process
  aborted after GUI exit with the same thread-termination message.

## Interpretation

This campaign does not support the hypothesis that concurrent control
systematically reduces video completion: the aggregate video result is 5/6 in
both isolated and combined modes. The per-parity directions conflict (FEC0 is
better combined; FEC1 is better isolated), which is consistent with random
loss and n=3 uncertainty rather than a stable interaction.

The stronger boundary observation is that the Targeted Arm/Takeoff/Land
sequence is already unreliable at 5% one-way loss without video. Concurrent
cells sometimes complete the sequence, so the evidence also does not show that
video traffic is required for control failure. This localizes the next
investigation to Targeted command delivery/automation and bounded timeout,
not stream buffer capacity.

FEC1 recovered chunks and video-only FEC1 completed 3/3, but n=3 is insufficient
to claim an FEC benefit. Spec 095 produced a different small-sample ordering;
both campaigns must remain visible.

## Limitations And Residual Risks

- Three repetitions permit descriptive localization only.
- MiniNDN random loss is not paired packet-for-packet across cells.
- Control-only is sequence-matched, not duration-matched.
- Two runs exposed a Ground Station joinable-thread/destructor abort after GUI
  exit. Both already failed their required components, so it does not change
  acceptance counts, but it is a real lifecycle defect requiring separate
  diagnosis.
- The mock flight controller, file H264 source, and host scheduling are not real
  radio/camera/PX4 validation.
- Command request-start timestamps are not currently emitted, so this campaign
  measures completion rather than per-command latency.
