# UAV MiniNDN Loss Campaign

## Command

```bash
python3 Experiments/NDNSF_UAV_Stream_Parity_Campaign.py \
  --out results/spec089-uav-stream-parity-loss5 \
  --runs 3 \
  --auto-stop-seconds 8
```

The tracked topology `Experiments/Topology/UAV_Stream_Parity_5pct.conf` uses
the same Memphis ground-station/controller and UCLA drone for every run with a
1 ms, 1000 Mbps link and 5% one-way packet loss. Video settings are fixed at
1200 kbps, width 320, file camera input, and the same eight-second stream.

## Results

| Run | Complete | FEC recovered | Max pending bytes | Max frame gap | RTT p50 | RTT p95 |
|---|---:|---:|---:|---:|---:|---:|
| 1 | yes | 3 | 14,400 | 0 | 47.0 ms | 120.0 ms |
| 2 | yes | 3 | 14,400 | 0 | 54.0 ms | 120.0 ms |
| 3 | yes | 1 | 21,600 | 0 | 59.5 ms | 120.0 ms |

All three runs remained below 48 pending chunks and 16 MiB pending bytes.
There were no unexpected stale-session or stale-stream packets. Stale rejection
itself is covered by the shared native parity vectors; the network campaign
confirms that valid current-session traffic is not falsely rejected.

Machine-readable local evidence:

```text
results/spec089-uav-stream-parity-loss5/campaign-summary.json
results/spec089-uav-stream-parity-loss5/campaign-summary.csv
```
